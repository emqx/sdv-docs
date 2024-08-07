# 监控

## 查询指标

sdv-flow 提供监控指标功能，可帮助用户实时了解 sdv-flow  的状态，并及时发现异常情况。云端 sdv-platform 有2种方式获取这些监控指标。

- 心跳上报

  每个sdv-flow 在注册成功后，会定时上报心跳信息到topic ecp/edgeagent/heartbeat，云端可通过订阅该topic 来获取信息。其中心跳信息的结构如下

  ```json
  {
    // vin码
    "vin": "ecs-2c63",  
     //流处理引擎状态
    "dataProcess": {  
      "status": 1,
      "cpuinfo": "50%",
      "memory": "100000000"
    },
    //消息总线状态
    "messageBus": {    
      "status": 1,
      "cpuinfo": "50%",
       "memory": "100000000"
    },
    //sdv-flow 内存及cpu
    "agent": {         
      "cpuinfo": "0.02%",
      "memory": "18006016"
    },
     //时间戳
    "ts": 1721722826,     
    //指标信息
    "metrics": "kuiper_rule_count{status=\"running\"} 1\nkuiper_rule_count{status=\"stop\"} 0\nkuiper_rule_status{ruleID=\"collect_parquet\"} 1\nnanomq_memory_usage 4034560\nnanomq_memory_usage_max 4034560\nnanomq_cpu_usage 0.10\nnanomq_cpu_usage_max 0.12\n"
  }
  ```

  其中metrics 就是指标信息，结构为指标数据以 \n 分割。
  
  指标信息的内容推送有2个模式：common，debug，common模式下默认只会推送常用的基础信息，debug模式下会推送所有的指标信息。默认为debug模式，如果当前默认模式下的指标不够，可以通过调sdv-platform的api 来切换模式。例如
  
  ```shell
  curl --location --request POST 'http://127.0.0.1:8082/api/metric/conf' \
  --header 'Content-Type: application/json' \
  --data-raw '{
  //vin 数组
      "vinIds":["ecs-2c63"], 
  // 模式 debug 或common    
      "level":"debug",
   //心跳上报中是否包括指标信息
      "enable":true
  }'
  ```
  
  sdv-platform  还集成了 [Prometheus](https://prometheus.io/docs/introduction/overview/)，用于收集和分析应用程序和服务的度量指标。心跳上报的内容通过emqx 推送到Prometheus，这种方式实现了实时的数据采集和分析，从而实现更精确的资源管理和性能调优，以及故障预测。
  
- api 查询

  车端 sdv-flow 在线的情况下，可以查询到所有的指标信息。${vin}表示vin 码。

  ```shell
  curl --location --request GET 'http://127.0.0.1:8082/api/metric/${vin}' 
  ```

  返回结果

  ```json
  {
    "metrics": [
      "kuiper_rule_count{status=\"running\"} 1",
      "kuiper_rule_count{status=\"stop\"} 0",
      "kuiper_rule_status{ruleID=\"collect_parquet\"} 1",
      "nanomq_memory_usage 4034560",
      "nanomq_memory_usage_max 4034560",
      "nanomq_cpu_usage 0.10",
      "nanomq_cpu_usage_max 0.12",
      "...",
    ],
    "ts": 0,
    "vin": "string"
  }
  ```

## 指标说明

| 指标名称                                   | 描述                                             |
| ------------------------------------------ | ------------------------------------------------ |
| nanomq_connections_count                   | Nanomq 连接数                                    |
| nanomq_connections_max                     | Nanomq 支持的最大连接数                          |
| nanomq_sessions_count                      | Nanomq 会话数                                    |
| nanomq_sessions_max                        | Nanomq 支持的最大会话数                          |
| nanomq_topics_count                        | Nanomq 主题数                                    |
| nanomq_topics_max                          | Nanomq 支持的最大主题数                          |
| nanomq_subscribers_count                   | Nanomq 订阅者数                                  |
| nanomq_subscribers_max                     | Nanomq 支持的最大订阅者数                        |
| nanomq_messages_received                   | Nanomq 接收到的消息计数                          |
| nanomq_messages_sent                       | Nanomq 发送的消息计数                            |
| nanomq_messages_dropped                    | Nanomq 丢弃的消息计数                            |
| nanomq_memory_usage                        | Nanomq CPU 使用率                                |
| nanomq_memory_usage_max                    | Nanomq 最大 CPU 使用率                           |
| nanomq_cpu_usage                           | Nanomq 内存使用量                                |
| nanomq_cpu_usage_max                       | Nanomq 最大内存使用量                            |
| kuiper_op_buffer_length                    | 所有 kuiper_op 实例共享的计划缓冲区长度          |
| kuiper_op_exceptions_total                 | kuiper_op 的用户异常总数                         |
| kuiper_op_process_latency_us               | kuiper_op 的最近一次处理的延时（毫秒）           |
| kuiper_op_process_latency_us_hist_bucket   | kuiper_op 的最近一次处理的延时历史数据（毫秒）   |
| kuiper_op_records_in_total                 | kuiper_op 操作接收的总消息数                     |
| kuiper_op_records_out_total                | kuiper_op 操作发布的总消息数                     |
| kuiper_sink_buffer_length                  | 所有 kuiper_sink 实例共享的计划缓冲区长度        |
| kuiper_sink_exceptions_total               | kuiper_sink 的用户异常总数                       |
| kuiper_sink_process_latency_us             | kuiper_sink 的最近一次处理延迟（毫秒）           |
| kuiper_sink_process_latency_us_hist        | kuiper_sink 的最近一次处理延迟历史数据（毫秒）   |
| kuiper_sink_records_in_total               | kuiper_sink 操作接收的总消息数                   |
| kuiper_sink_records_out_total              | kuiper_sink 操作发布的总消息数                   |
| kuiper_source_buffer_length                | 所有 kuiper_source 实例共享的计划缓冲区长度      |
| kuiper_source_exceptions_total             | kuiper_source 的用户异常总数                     |
| kuiper_source_process_latency_us           | kuiper_source 的最近一次处理延迟（毫秒）         |
| kuiper_source_process_latency_us_hist      | kuiper_source 的最近一次处理延迟历史数据（毫秒） |
| kuiper_source_records_in_total             | kuiper_source 操作接收的总消息数                 |
| kuiper_source_records_out_total            | kuiper_source 操作发布的总消息数                 |
| process_cpu_seconds_total                  | 进程花费的总用户和系统 CPU 时间（秒）            |
| process_virtual_memory_bytes               | 虚拟内存大小（字节）                             |
| process_virtual_memory_max_bytes           | 可用的最大虚拟内存量（字节）                     |
| process_resident_memory_bytes              | 常驻内存大小（字节）                             |
| process_start_time_seconds                 | 进程自 Unix 纪元以来的开始时间（秒）             |
| process_max_fds                            | 最大打开文件描述符数量                           |
| process_open_fds                           | 打开的文件描述符数量                             |
| promhttp_metric_handler_requests_in_flight | 当前正在服务的抓取请求数量                       |
| promhttp_metric_handler_requests_total     | 按 HTTP 状态码统计的总抓取请求数                 |
| kuiper_rule_count                          | kuiper 中的规则数量                              |
| kuiper_rule_status                         | kuiper 中的规则状态                              |
| messages_processed_total                   | kuiper 处理的总消息数                            |

