# 云边协同

sdv-platform 可用于车辆注册、管理、发布规则、批量操作和监控，这篇文章详细介绍了一个简单的 demo，使用api 配置来实现数据的采集，处理以及日志文件的上传。

## 1.部署

首先请参考 [快速入门](../quick-start/quick-start.md)，部署云端，以及一个车端 sdv-flow，其中车端sdv-flow 启动会自动生成 vin 码，来作为车端唯一且不变的标识。该demo中的 sdv-flow 的vin 码为 DV_PV_TASK_ENABLE。

sdv-flow 启动成功后会自动向云端发起注册，在云端license 充足的情况下，会注册成功。接下来便可以在云端对车端 sdv-flow 进行控制。

## 2.下发规则

### 2.1创建CANUDP 流

```json
curl --location --request POST 'http://127.0.0.1:8082/api/edgeservice/proxy/DV_PV_TASK_ENABLE/streams' \
--header '_emqx_request_id_: demotest' \
--header 'Content-Type: application/json' \
--data-raw '{
    "sql": "CREATE STREAM canudp () WITH (TYPE=\"mqtt\", CONF_KEY=\"default\", FORMAT=\"canjson\", SHARED=\"TRUE\", SCHEMAID=\"/tmp/dbc\", DATASOURCE=\"canudp\")"
}'
```

api 返回结果即为下发结果，另外 用户可通过 订阅 emqx 的 topic `agent/{vin}/proxy/ack/{request_id}`  的方式来获取下发结果，其中 `request_id` 可通过 在header 中设置 `_emqx_request_id_` 的值， 若未设置，则会随机生成。

### 2.2 创建规则0：Collectall

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/edgeservices/config' \
--header '_emqx_request_id_prefix_: 1111--' \
--header 'Content-Type: application/json' \
--data-raw '{
    "ids": [
        "DV_PV_TASK_ENABLE"
    ],
    "method": "POST",
    "path": "/rules",
    "body": "{\"id\":\"collectall\",\"sql\":\"SELECT * FROM canudp\",\"actions\":[{\"mqtt\":{\"sendSingle\":true,\"server\":\"tcp://127.0.0.1:1883\",\"topic\":\"results/all\"}}]}"
}'
```

该api 适用于对多个车端下发相同的配置，其中 ids 表示vin 码数组。另外 用户可通过 订阅 emqx 的 topic `agent/{vin}/proxy/ack/{request_id}`  的方式来获取下发结果，header 中设置的`_emqx_request_id_prefix_` 的值即为前缀，`{request_id}`的值为前缀加序号生成， 若未设置，则会随机生成。

返回结果格式为 `{"id": "7527e427"}`，id 即为本次下发的任务id，用户可通过 api 查询下发结果。

```shell
curl --location --request GET 'http://127.0.0.1:8082/api/edgeservices/config/result?id=7527e427'
```

### 2.3 创建规则1：CAN 数据流式采集（每10秒聚合一次）

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/edgeservices/config' \
--header '_emqx_request_id_prefix_: 1111--' \
--header 'Content-Type: application/json' \
--data-raw '{
    "ids": [
        "DV_PV_TASK_ENABLE"
    ],
    "method": "POST",
    "path": "/rules",
    "body":	"{\n  \"id\": \"collect1\",\n  \"sql\": \"SELECT last_value(ENGINE.RPM,true) as RPM_10s, last_value(ENGINE.Temperature,true) as Temperature_10s, last_value(ENGINE.Torque,true) as Torque_10s FROM canudp Group BY TumblingWindow(ss,10)\",\n  \"actions\": [{\n    \"mqtt\": {\n      \"sendSingle\": false,\n      \"server\": \"tcp://127.0.0.1:1883\",\n      \"topic\": \"results/collect1\"\n    }\n  }]\n}"
}'
```

###  2.4 创建规则2：CAN 数据触发式窗口批式采集

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/edgeservices/config' \
--header '_emqx_request_id_prefix_: 1111--' \
--header 'Content-Type: application/json' \
--data-raw '{
    "ids": [
        "DV_PV_TASK_ENABLE"
    ],
    "method": "POST",
    "path": "/rules",
    "body": "{\n  \"id\": \"collect_batch\",\n  \"sql\": \"SELECT DRIVE.Speed as Speed, DRIVE.Acceleration as Acceleration, DRIVE.Angle as Angle, PEDAL.BrakePosition as BrakePosition FROM canudp Group BY SlidingWindow(ss, 1, 1) OVER (WHEN latest(PEDAL.BrakePosition)>= 10)\",\n  \"actions\": [\n    {\n      \"file\": {\n        \"compression\": \"zstd\",\n        \"delete\": 7200,\n        \"fileType\": \"lines\",\n        \"format\": \"json\",\n        \"interval\": 100,\n        \"omitIfEmpty\": true,\n        \"path\": \"/batchfile/can-json-poc.zst\",\n        \"rollingCount\": 2,\n        \"rollingInterval\": 0,\n        \"rollingNamePattern\": \"suffix\",\n        \"segmentSize\": 1025,\n        \"sendSingle\": false\n      }\n    }\n  ]\n}"
}'
```

### 2.5 创建规则3：CAN 数据触发式全量落盘数据采集

#### 2.5.1 创建内存流

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/edgeservices/config' \
--header 'Content-Type: application/json' \
--data-raw '{
    "ids": [
        "DV_PV_TASK_ENABLE"
    ],
    "method": "POST",
    "path": "/streams",
    "body": "{\"sql\": \"CREATE STREAM triggerStream() WITH (TYPE=\\\"memory\\\", FORMAT=\\\"json\\\", DATASOURCE=\\\"trigger\\\")\"}"
}'
```

#### 2.5.2.创建通用落盘规则

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/edgeservices/config' \
--header 'Content-Type: application/json' \
--data-raw '{
    "ids": [
        "DV_PV_TASK_ENABLE"
    ],
    "method": "POST",
    "path": "/rules",
    "body": "{\"id\":\"ruleTrigger\",\"sql\":\"SELECT \\\"EX2NANO\\\" as id, \\\"search\\\" as cmd, dedup_trigger(begin, finish, ts, 999999) as ranges, ruleid FROM triggerStream WHERE isNull(ruleid) = false\",\"actions\":[{\"nano\":{\"url\":\"ipc:///tmp/nanomq_hook.ipc\",\"sendSingle\":true}},{\"mqtt\":{\"server\":\"tcp://127.0.0.1:1883\",\"topic\":\"result/trigger\",\"sendSingle\":true}}]}"
}'
```

#### 2.5.3 创建触发落盘规则

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/edgeservices/config' \
--header 'Content-Type: application/json' \
--data-raw '{
    "ids": [
        "DV_PV_TASK_ENABLE"
    ],
    "method": "POST",
    "path": "/rules",
    "body": "{\"id\":\"collect_parquet\",\"sql\":\"SELECT event_time() - 60000 as begin, event_time() as finish, event_time() as ts, rule_id() as ruleid FROM canudp WHERE PEDAL.BrakePosition >= 10\",\"actions\":[{\"memory\":{\"topic\":\"trigger\",\"sendSingle\":true}},{\"mqtt\":{\"server\":\"tcp://127.0.0.1:1883\",\"topic\":\"result/trigger\",\"sendSingle\":true}}]}"
}'
```

## 3. 模拟数据流入

### 3.1 流入数据模拟例子

```json
{"frames":[{"id":0,"data":"AA1709B8F2DFA3F0","bus":4,"d":0,"t":0},{"id":1,"data":"BB281AC903E0B401","bus":4,"d":0,"t":0},{"id":2,"data":"CC392BDA14F1C512","bus":4,"d":0,"t":0},{"id":3,"data":"DD4A3CEB2502D623","bus":4,"d":0,"t":0},{"id":4,"data":"EE5B4DFC3613E734","bus":4,"d":0,"t":0},{"id":5,"data":"FF6C5E0D4724F845","bus":4,"d":0,"t":0},{"id":6,"data":"007D6F1E58350956","bus":4,"d":0,"t":0},{"id":7,"data":"118E702F69461A67","bus":4,"d":0,"t":0},{"id":8,"data":"229F81307A572B78","bus":4,"d":0,"t":0},{"id":9,"data":"33A092418B683C89","bus":4,"d":0,"t":0},{"id":10,"data":"44B1A3529C794D9A","bus":4,"d":0,"t":0},{"id":11,"data":"55C2B463AD8A5EAB","bus":4,"d":0,"t":0},{"id":12,"data":"66D3C574BE9B6FBC","bus":4,"d":0,"t":0},{"id":13,"data":"77E4D685CFAC70CD","bus":4,"d":0,"t":0},{"id":14,"data":"88F5E796D0BD81DE","bus":4,"d":0,"t":0},{"id":15,"data":"9906F8A7E1CE92EF","bus":4,"d":0,"t":0},{"id":16,"data":"AA1709B8F2DFA3F0","bus":4,"d":0,"t":0},{"id":17,"data":"BB281AC903E0B401","bus":4,"d":0,"t":0},{"id":18,"data":"CC392BDA14F1C512","bus":4,"d":0,"t":0},{"id":19,"data":"DD4A3CEB2502D623","bus":4,"d":0,"t":0},{"id":20,"data":"EE5B4DFC3613E734","bus":4,"d":0,"t":0},{"id":21,"data":"FF6C5E0D4724F845","bus":4,"d":0,"t":0},{"id":22,"data":"007D6F1E58350956","bus":4,"d":0,"t":0}]}
```

### 3.2 模拟数据生成脚本

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

## 4.查询车端文件列表

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/edgeagents/DV_PV_TASK_ENABLE/filelist' \
--header 'Content-Type: application/json' \
--data-raw '{
    "paths": [
        "/usrdata/edgeagent/"
    ]
}'
```

## 5.日志采集

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/edgeagents/DV_PV_TASK_ENABLE/fileupload' \
--header 'Content-Type: application/json' \
--data-raw '{
    "request_id": "20240509",
    "files": [
        "/usrdata/edgeagent/kuiper/log/stream.log"
    ],
    "filenames": [
        "stream.log"
    ],
    "delete":[-1]
}'
```

车端生成日志/opt/sdv-flow/software/ekuiper/log/stream.log，调api，车端将把 stream.log 日志文件上报到云端，用户可通过订阅emqx topic `file_transfer/#`  获取日志。