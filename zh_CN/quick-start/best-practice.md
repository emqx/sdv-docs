# 最佳实践
这篇文章详细介绍了一个数据闭环场景中断触发式采集数据的demo，并提供了参数调整的思路。

## 1. 在流处理引擎中创建流
流处理引擎的功能主要由源（Source），规则（Rule）和目标（Sink）三个基本元素组成，复杂的流功能都是由这些基本元素组合而成。
### 1.1 创建 CANUDP 流

要处理车内数据流，首先我们创建一个专门解码处理 CAN 协议的 CANUDP 流。
使用HTTP 接口可以快速创建规则流，图示为创建一个车辆数据源：CAN-UDP数据总线流，并得到成功的返回结果表示流创建成功。

```bash
curl http://127.0.0.1:14260/streams -d '{"sql": "CREATE STREAM canudp () WITH (TYPE=\"mqtt\", CONF_KEY=\"default\", FORMAT=\"canjson\", SHARED=\"TRUE\", SCHEMAID=\"/tmp/dbc\", DATASOURCE=\"canudp\")"}'
```

### 1.2 创建规则0：Collectall

再使用HTTP API 为 CANUDP 流创建数据源，以MQTT数据源为例，订阅的本地主题是”result/all”。
```bash
curl http://127.0.0.1:14260/rules -d '{"id":"collectall","sql":"SELECT * FROM canudp","actions":[{"mqtt":{"sendSingle":true,"server":"tcp://127.0.0.1:1883","topic":"results/all"}}]}'
```
如此所有本地数据总线发布到 ”result/all” 主题的数据都会流入该条规则。

### 1.3 创建规则1：CAN 数据流式采集（每10秒聚合一次）
有了数据源后，继续创建采集规则，此处以降频规则为例，用时间窗口汇聚 "results/collect1" 主题上的 10s数据，并筛选信号。

```bash
curl -X POST http://127.0.0.1:14260/rules -d '{
  "id": "collect1",
  "sql": "SELECT last_value(ENGINE.RPM,true) as RPM_10s, last_value(ENGINE.Temperature,true) as Temperature_10s, last_value(ENGINE.Torque,true) as Torque_10s FROM canudp Group BY TumblingWindow(ss,10)",
  "actions": [{
    "mqtt": {
      "sendSingle": false,
      "server": "tcp://127.0.0.1:1883",
      "topic": "results/collect1"
    }
  }]
}'
```

### 1.4 创建规则2：CAN 数据触发式窗口批式采集

再用HTTP API建立滑动窗口来解析窗口内数据进行模式匹配完成事件触发式采集。

```bash
curl http://127.0.0.1:14260/rules -d '{
  "id": "collect_batch",
  "sql": "SELECT DRIVE.Speed as Speed, DRIVE.Acceleration as Acceleration, DRIVE.Angle as Angle, PEDAL.BrakePosition as BrakePosition FROM canudp Group BY SlidingWindow(ss, 1, 1) OVER (WHEN latest(PEDAL.BrakePosition)>= 10)",
  "actions": [
    {
      "file": {
        "compression": "zstd",
        "delete": 7200,
        "fileType": "lines",
        "format": "json",
        "interval": 100,
        "omitIfEmpty": true,
        "path": "/batchfile/can-json-poc.zst",
        "rollingCount": 2,
        "rollingInterval": 0,
        "rollingNamePattern": "suffix",
        "segmentSize": 1025,
        "sendSingle": false
      }
    }
  ]
}'
```
如此，流处理引擎会自动匹配刹车制动场景，然后将命中该事件前后100s的时间窗口内的数据进行压缩打包上传。

### 1.5 创建规则3：CAN 数据触发式全量落盘数据采集
在需要进行整车仿真的时候，和部分复杂场景下，我们无法预先制定该场景所需要的信号矩阵来进行埋点数据采集。这种情况建议使用场景触发式的全量原始数据采集。如此避免了对所有整车信号的解析带来无法承受的性能消耗，又保证了数据的完整性。

#### 1.5.1 创建内存流
首先创建一个内部内存流用于记录场景匹配和触发事件。此段我们建议创建内存流来汇聚所有的触发结果。但请注意此步骤并非必须。每个规则仍然都可以不经过事件汇聚而独立发出触发事件。

```bash
curl http://127.0.0.1:14260/streams -d "{\"sql\": \"CREATE STREAM triggerStream() WITH (TYPE=\\\"memory\\\", FORMAT=\\\"json\\\", DATASOURCE=\\\"trigger\\\")\"}"
```

#### 1.5.2 创建通用落盘规则
创建通用落盘规则，操作 triggerStream 流，用于接收触发落盘的消息，去重并转为前置消息队列服务所需要的格式。用于从消息队列中查询目标时间段的全量原始数据。

```bash
curl http://127.0.0.1:14260/rules -d '{
  "id": "ruleTrigger",
  "sql": "SELECT \"EX2NANO\" as id, \"search\" as cmd, dedup_trigger(begin, finish, ts, 999999) as ranges, ruleid FROM triggerStream WHERE isNull(ruleid) = false",
  "actions": [
    {
      "nano": {
        "url": "ipc:///tmp/nanomq_hook.ipc",
        "sendSingle": true
      }
    },
    {
      "mqtt": {
        "server": "tcp://127.0.0.1:1883",
        "topic": "result/trigger",
        "sendSingle": true
      }
    }
  ]
}'
```

#### 1.5.3 创建触发落盘规则
此步骤与创建的内部内存流对应，用于在定义的事件范围内进行触发信号去重。避免配置的多规则流水线同时命中的时候进行重复的全量整车数据搜索和上传。
```bash
curl http://127.0.0.1:14260/rules -d '{
  "id": "collect_parquet",
  "sql": "SELECT event_time() - 60000 as begin, event_time() as finish, event_time() as ts, rule_id() as ruleid FROM canudp WHERE ts - last_hit_time() > 10000",
  "actions": [
    {
      "memory": {
        "topic": "trigger",
        "sendSingle": true
      }
    },
    {
      "mqtt": {
        "server": "tcp://127.0.0.1:1883",
        "topic": "result/trigger",
        "sendSingle": true
      }
    }
  ]
}'
```

## 2. 模拟数据流入

以自定义的模拟数据源为例，数据源会流入消息队列和数据总线，滚动落盘后经由流处理引擎模块中的规则触发查询后上传。

<img width="416" alt="image" src="https://github.com/user-attachments/assets/32a22dc4-3529-46b2-be19-f61777d37173">

### 2.1 流入数据模拟例子

```json
{"frames":[{"id":0,"data":"AA1709B8F2DFA3F0","bus":4,"d":0,"t":0},{"id":1,"data":"BB281AC903E0B401","bus":4,"d":0,"t":0},{"id":2,"data":"CC392BDA14F1C512","bus":4,"d":0,"t":0},{"id":3,"data":"DD4A3CEB2502D623","bus":4,"d":0,"t":0},{"id":4,"data":"EE5B4DFC3613E734","bus":4,"d":0,"t":0},{"id":5,"data":"FF6C5E0D4724F845","bus":4,"d":0,"t":0},{"id":6,"data":"007D6F1E58350956","bus":4,"d":0,"t":0},{"id":7,"data":"118E702F69461A67","bus":4,"d":0,"t":0},{"id":8,"data":"229F81307A572B78","bus":4,"d":0,"t":0},{"id":9,"data":"33A092418B683C89","bus":4,"d":0,"t":0},{"id":10,"data":"44B1A3529C794D9A","bus":4,"d":0,"t":0},{"id":11,"data":"55C2B463AD8A5EAB","bus":4,"d":0,"t":0},{"id":12,"data":"66D3C574BE9B6FBC","bus":4,"d":0,"t":0},{"id":13,"data":"77E4D685CFAC70CD","bus":4,"d":0,"t":0},{"id":14,"data":"88F5E796D0BD81DE","bus":4,"d":0,"t":0},{"id":15,"data":"9906F8A7E1CE92EF","bus":4,"d":0,"t":0},{"id":16,"data":"AA1709B8F2DFA3F0","bus":4,"d":0,"t":0},{"id":17,"data":"BB281AC903E0B401","bus":4,"d":0,"t":0},{"id":18,"data":"CC392BDA14F1C512","bus":4,"d":0,"t":0},{"id":19,"data":"DD4A3CEB2502D623","bus":4,"d":0,"t":0},{"id":20,"data":"EE5B4DFC3613E734","bus":4,"d":0,"t":0},{"id":21,"data":"FF6C5E0D4724F845","bus":4,"d":0,"t":0},{"id":22,"data":"007D6F1E58350956","bus":4,"d":0,"t":0}]}
```

### 2.2 模拟数据生成脚本

此处提供一个模拟数据输入的脚步代码，供用户验证。
```c
// input_zegote.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_FRAME_SIZE 10000

void generate_hex_string(char *str, int j) {
    const char *hex_chars = "0123456789ABCDEF";
    int i;

    srand(time(NULL));

    for (i = 0; i < 16; ++i) {
        str[i] = hex_chars[(rand() + j) % 16];
    }   

    str[16] = '\0';
}

int main(void)
{
    FILE *fp;
    fp = fopen("/tmp/input.txt", "w");
    if (fp == NULL) {
        printf("Error opening file\n");
        exit(1);
    }   
    fprintf(fp, "{\"frames\":[");
    for (int i = 0; i < MAX_FRAME_SIZE; i++) {
        char frame[100];
        memset(frame, 0, sizeof(frame));
        strcat(frame, "{\"id\":");
        char id[10];
        sprintf(id, "%d", i); 
        strcat(frame, id);
        strcat(frame, ",\"data\":\"");
        char hex_string[17];
        generate_hex_string(hex_string, i); 
        strcat(frame, hex_string);
        strcat(frame, "\",\"bus\":4,\"d\":0,\"t\":0}");
        if (i != MAX_FRAME_SIZE - 1)
            strcat(frame, ",");
        fprintf(fp, frame);
    }   
    fprintf(fp, "]}");
    return 0;
}
```

## 3. 消息队列模块配置文件最佳实践

### 3.1 发送频率
Ringbus 是消息队列模块中 MQTT-Stream 功能内置的缓存队列，用于进行批量刷盘前的消息合并。
例如 ringbus 的 cap 设置为 2000，消息发送频率为 10 毫秒，那么每 10ms * 2000 = 20 秒执行一次落盘动作。
消息聚合是一个优化选项，可以将 10 毫秒内的消息聚合为一条消息发送，以降低I/O数量，以减低CPU负载和提高性能。建议根据存储介质和运行硬件平台来适配这些参数。

### 3.2 配置相关
#### 3.2.1 Ringbus
在消息总线模块配置中，Ringbus 的 cap 设置不宜过大或过小。cap 和消息大小会影响落盘文件的大小和压缩倍率。例如，如果一条消息大小为 10KB，cap 设置为 2000，那么一次落盘的数据量为 20MB。进行刷盘前会使用选择的 GZIP 算法进行压缩，此算法常见压缩倍率为3-5 倍。

```ini
exchange_client.mq1 {
    ...
    exchange {
        ...
        ringbus = {
            ...
            compress = gzip
            cap = 2000
            ...
        }
    }
}
```

#### 3.3.2 Parquet
在消息总线模块配置中，Parquet 的 file_size 应尽可能大，最好能包含一次落盘的数据量。例如，如果一次落盘约为 20MB，那么 file_size 的限制可以配置为 50MB。否则，Parquet 会对文件进行切分，增加额外的性能开销和磁盘写入寿命消耗。
而在对单个落盘文件有限制需求的情况下，也可以通过此选项来限制每个分区落盘文件大小，来达到切片处理的效果。

```ini
parquet {
    ...
    # 默认值: 10M
    # 值: 数字
    # 支持单位: KB | MB | GB
    file_size = 50MB
    ...
}
```
