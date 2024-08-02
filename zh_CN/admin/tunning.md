# 性能调优指南

本章节提供一些常见的性能调优方案和自限性资源隔离方法。用户可根据提示自行针对特殊应用场景进行对应的配置修改。

## 消息总线模块调优
消息总线是 sdv-flow 中用于提供数据的可互操作性的基础服务，其包括了 MQTT 消息服务和消息队列功能。

### MQTT 消息服务调优

sdv-flow 均为多线程程序，其中的 MQTT 消息服务也不例外。其内建 Actor 编程模型并行化计算负载，内置全异步 I/O 框架，能够最大化利用多核能力。在此基础上我们针对 Linux 平台和 MQTT 协议优化后，成功将负载平均分配到每个 CPU 核。因此 NanoMQ 在现代 SMP 系统能够以更少的 CPU 资源占用去承担高达一百万每秒的消息吞吐压力。

而强大的消息吞吐能力，可能会受到消费端能力影响而导致消息积压，内存增长。为此可以通过限制内部物理线程和逻辑线程数量来控制。

```
system {
  // 物理线程数量，可以根据运行平台的 CPU 核心数和超线程能力配置。
  // 例如运行于 aarch64 Cortex A53 8 core 上时，应设置成 8 来获得最大的吞吐能力。也可以设置成 4 来限制其在最大负载压力下的 CPU 占用。（设置成 4 即最大占用400%， 设置成 8 就是最大占用400%，即全核占用）
	num_taskq_thread = 0
	max_taskq_thread = 0
  // 逻辑线程数量，显著影响内存占用。决定了同一时间能够并行并发处理的最大逻辑操作数量。
  // 每个消息的接收订阅，内部钩子函数，落盘存储，发布投递等具有明确前后上下文顺序需要保存的操作即视为需要占用一个逻辑线程。
  // 一般建议设置成物理线程的 2-4 倍。若内存不足，也可以配置更小，但随之带来丢失消息和会话积压的风险。 
	parallel = 0
}
```

MQTT 协议调优
```
mqtt {
	# # max_packet_size
	max_packet_size = 1KB
	
	# # max_mqueue_len
	max_mqueue_len = 2048

	
	# # retry_interval (s)
	retry_interval = 10s
	
	# # The backoff for MQTT keepalive timeout.
	# # broker will discolse client when there is no activity for
	# # 'Keepalive * backoff * timeout.
	# #
	# # Hot updatable
	# # Value: Float > 0.5
	keepalive_multiplier = 1.25
	
}
```
