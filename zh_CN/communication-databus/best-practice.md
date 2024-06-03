# 最佳实践
这篇文章详细介绍了一个简单的demo，并提供了参数调整的思路，以优化整个链路的性能和可用性。

## 1. Ekuiper 创建流
### 1.1 创建 CANUDP 流
```bash
curl http://127.0.0.1:9081/streams -d '{"sql": "CREATE STREAM canudp () WITH (TYPE=\"mqtt\", CONF_KEY=\"default\", FORMAT=\"canjson\", SHARED=\"TRUE\", SCHEMAID=\"/tmp/dbc\", DATASOURCE=\"canudp\")"}'
```

### 1.2 创建规则0：Collectall
```bash
curl http://127.0.0.1:9081/rules -d '{"id":"collectall","sql":"SELECT * FROM canudp","actions":[{"mqtt":{"sendSingle":true,"server":"tcp://127.0.0.1:1883","topic":"results/all"}}]}'
```

### 1.3 创建规则1：CAN 数据流式采集（每10秒聚合一次）
```bash
curl -X POST http://127.0.0.1:9081/rules -d '{
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
```bash
curl http://127.0.0.1:9081/rules -d '{
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

### 1.5 创建规则3：CAN 数据触发式全量落盘数据采集
#### 1.5.1 创建内存流
```bash
curl http://127.0.0.1:9081/streams -d "{\"sql\": \"CREATE STREAM triggerStream() WITH (TYPE=\\\"memory\\\", FORMAT=\\\"json\\\", DATASOURCE=\\\"trigger\\\")\"}"
```

#### 1.5.2 创建通用落盘规则
```bash
curl http://127.0.0.1:9081/rules -d '{
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
```bash
curl http://127.0.0.1:9081/rules -d '{
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
### 2.1 流入数据模拟例子
```json
{"frames":[{"id":0,"data":"AA1709B8F2DFA3F0","bus":4,"d":0,"t":0},{"id":1,"data":"BB281AC903E0B401","bus":4,"d":0,"t":0},{"id":2,"data":"CC392BDA14F1C512","bus":4,"d":0,"t":0},{"id":3,"data":"DD4A3CEB2502D623","bus":4,"d":0,"t":0},{"id":4,"data":"EE5B4DFC3613E734","bus":4,"d":0,"t":0},{"id":5,"data":"FF6C5E0D4724F845","bus":4,"d":0,"t":0},{"id":6,"data":"007D6F1E58350956","bus":4,"d":0,"t":0},{"id":7,"data":"118E702F69461A67","bus":4,"d":0,"t":0},{"id":8,"data":"229F81307A572B78","bus":4,"d":0,"t":0},{"id":9,"data":"33A092418B683C89","bus":4,"d":0,"t":0},{"id":10,"data":"44B1A3529C794D9A","bus":4,"d":0,"t":0},{"id":11,"data":"55C2B463AD8A5EAB","bus":4,"d":0,"t":0},{"id":12,"data":"66D3C574BE9B6FBC","bus":4,"d":0,"t":0},{"id":13,"data":"77E4D685CFAC70CD","bus":4,"d":0,"t":0},{"id":14,"data":"88F5E796D0BD81DE","bus":4,"d":0,"t":0},{"id":15,"data":"9906F8A7E1CE92EF","bus":4,"d":0,"t":0},{"id":16,"data":"AA1709B8F2DFA3F0","bus":4,"d":0,"t":0},{"id":17,"data":"BB281AC903E0B401","bus":4,"d":0,"t":0},{"id":18,"data":"CC392BDA14F1C512","bus":4,"d":0,"t":0},{"id":19,"data":"DD4A3CEB2502D623","bus":4,"d":0,"t":0},{"id":20,"data":"EE5B4DFC3613E734","bus":4,"d":0,"t":0},{"id":21,"data":"FF6C5E0D4724F845","bus":4,"d":0,"t":0},{"id":22,"data":"007D6F1E58350956","bus":4,"d":0,"t":0}]}
```

### 2.2 模拟数据生成脚本
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

## 3. 配置文件最佳实践

### 3.1 发送频率
建议消息间隔不要超过10毫秒。如果 ringbus 的 cap 设置为 2000，消息发送频率为 10 毫秒，那么每 20 秒执行一次落盘动作。消息聚合是一个优化选项，可以将 10 毫秒内的消息聚合为一条消息发送，以降低负载和提高性能。

### 3.2 配置相关
#### 3.2.1 Ringbus
在 Naomq 配置中，Ringbus 的 cap 设置不宜过大或过小。cap 和消息大小会影响落盘文件的大小。例如，如果一条消息大小为 10KB，cap 设置为 2000，那么一次落盘的数据量为 20MB。

```ini
exchange_client.mq1 {
    ...
    exchange {
        ...
        ringbus = {
            ...
            cap = 2000
            ...
        }
    }
}
```

#### 3.3.2 Parquet
在 Nanomq 配置中，Parquet 的 file_size 应尽可能大，最好能包含一次落盘的数据量。例如，如果一次落盘约为 20MB，那么 file_size 的限制可以配置为 50MB。否则，Parquet 会对文件进行切分，增加额外的性能开销。

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
