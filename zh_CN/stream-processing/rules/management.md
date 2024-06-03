# 规则管理

用户可以使用 REST API 进行规则的管理，例如创建、显示、删除、描述、启动、停止和重新启动规则。

## 规则增删改查

### 创建规则

该 API 接受 JSON 内容并创建和启动规则。

```shell
POST http://localhost:9081/rules
```

请求示例：

```json
{
  "id": "rule1",
  "sql": "SELECT * FROM demo",
  "actions": [{
    "log":  {}
  }]
}
```

### 展示规则

该 API 用于显示服务器中定义的所有规则和简要状态描述。

```shell
GET http://localhost:9081/rules
```

响应示例：

```json
[
  {
    "id": "rule1",
    "status": "Running"
  },
  {
     "id": "rule2",
     "status": "Stopped: canceled by error."
  }
]
```

### 描述规则

该 API 用于打印规则的详细定义。

```shell
GET http://localhost:9081/rules/{id}
```

路径参数  `id` 是规则的 id 或名称。

响应示例：

```json
{
  "sql": "SELECT * from demo",
  "actions": [
    {
      "log": {}
    },
    {
      "mqtt": {
        "server": "tcp://127.0.0.1:1883",
        "topic": "demoSink"
      }
    }
  ]
}
```

### 更新规则

该 API 接受 JSON 内容并更新规则。

```shell
PUT http://localhost:9081/rules/{id}
```

路径参数  `id` 是原有规则的 id 或名称。

请求示例：

```json
{
  "id": "rule1",
  "sql": "SELECT * FROM demo",
  "actions": [{
    "log":  {}
  }]
}
```

### 删除规则

该 API 用于删除规则。

```shell
DELETE http://localhost:9081/rules/{id}
```

## 规则启停

### 启动规则

该 API 用于开始运行规则。

```shell
POST http://localhost:9081/rules/{id}/start
```

### 停止规则

该 API 用于停止运行规则。

```shell
POST http://localhost:9081/rules/{id}/stop
```

## 规则监控

### 获取规则的状态

该命令用于获取规则的状态。 如果规则正在运行，则将实时检索状态指标。 状态可以是：

- $metrics
- 停止： $reason

```shell
GET http://localhost:9081/rules/{id}/status
```

响应示例：

```shell
{
    "source_demo_0_records_in_total":5,
    "source_demo_0_records_out_total":5,
    "source_demo_0_exceptions_total":0,
    "source_demo_0_process_latency_ms":0,
    "source_demo_0_buffer_length":0,
    "source_demo_0_last_invocation":"2020-01-02T11:28:33.054821",
    ...
    "op_filter_0_records_in_total":5,
    "op_filter_0_records_out_total":2,
    "op_filter_0_exceptions_total":0,
    "op_filter_0_process_latency_ms":0,
    "op_filter_0_buffer_length":0,
    "op_filter_0_last_invocation":"2020-01-02T11:28:33.054821",
    ...
}
```

运行指标主要包括两个部分，一部分是 status，用于标示规则是否正常运行，其值可能为 `running`，`stopped manually` 等。另一部分为规则每个算子的运行指标。规则的算子根据规则的 SQL 生成，每个规则可能会有所不同。在此例中，规则 SQL 为最简单的 `SELECT * FROM demo`，action 为 MQTT，其生成的算子为 [source_demo，op_project，sink_mqtt] 3个。每一种算子都有相同数目的运行指标，与算子名字合起来构成一条指标。例如，算子 source_demo_0 的输入数量 records_in_total 的指标为 `source_demo_0_records_in_total`。

### 运行指标

每个算子的运行指标是相同的，主要有以下几种：

- records_in_total：读入的消息总量，表示规则启动后处理了多少条消息。
- records_out_total：输出的消息总量，表示算子**正确**处理的消息数量。
- process_latency_us：最近一次处理的延时，单位为微妙。该值为瞬时值，可了解算子的处理性能。整体规则的延时一般由延时最大的算子决定。
- buffer_length：算子缓冲区长度。由于算子之间计算速度会有差异，各个算子之间都有缓冲队列。缓冲区长度较大的话说明算子处理较慢，赶不上上游处理速度。
- last_invocation：算子的最后一次运行的时间。
- exceptions_total：异常总量。算子运行中产生的非不可恢复的错误，例如连接中断，数据格式错误等均计入异常，而不会中断规则。
- last_exception：最近一次的异常的错误信息。
- last_exception_time：最近一次异常的发生时间。

这些运行指标中的数值类型指标均可使用 Prometheus 进行监控。

## 规则导入导出

eKuiper REST api 允许您导入导出当前的所有流和规则配置。

## 规则集格式

导入导出规则集的文件格式为 JSON，其中可包含三个部分：流 `streams`，表 `tables` 和规则 `rules`。每种类型保存名字和创建语句的键值对。在以下示例文件中，我们定义了一个流和两条规则。

```json
{
    "streams": {
        "demo": "CREATE STREAM demo () WITH (DATASOURCE=\"users\", FORMAT=\"JSON\")"
    },
    "tables": {},
    "rules": {
        "rule1": "{\"id\": \"rule1\",\"sql\": \"SELECT * FROM demo\",\"actions\": [{\"log\": {}}]}",
        "rule2": "{\"id\": \"rule2\",\"sql\": \"SELECT * FROM demo\",\"actions\": [{  \"log\": {}}]}"
    }
}
```

## 导入规则集

该 API 接受规则集并将其导入系统中。若规则集中的流或规则已存在，则不再创建。导入的规则将立刻启动。API 返回文本告知创建的流和规则的数目。 API 支持通过文本内容或者文件 URI 的方式指定规则集。

示例1：通过文本内容导入

```shell
POST http://{{host}}/ruleset/import
Content-Type: application/json

{
  "content": "$规则集 json 内容"
}
```

示例2：通过文件 URI 导入

```shell
POST http://{{host}}/ruleset/import
Content-Type: application/json

{
  "file": "file:///tmp/a.json"
}
```

## 导出规则集

导出 API 返回二进制流，在浏览器使用时，可选择下载保存的文件路径。

```shell
POST http://{{host}}/ruleset/export
```
