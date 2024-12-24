# 环形队列配置

## 概述

本文档介绍了环形队列（Ringbus）的配置方法及相关选项。环形队列用于存储MQTT消息，并可以配置其容量和满时的处理行为。详细配置如下：

### exchange_client

`exchange_client` 是承载多个 `exchange` 的客户端。

### exchange

`exchange` 用于接收特定主题（topic）的消息，并将消息数据放入 `ringbus` 中。其配置如下：

```hocon
exchange_client.mq1 {
    # exchanges contains multiple MQ exchanger
    exchange {
        # MQTT Topic for filtering messages and saving to queue
        topic = "exchange/topic1",
        # MQ name
        name = "exchange_no1",
        # MQ category. Only support Ringbus for now
        ringbus = {
            # ring buffer name
            name = "ringbus",
            # max length of ring buffer (msg count)
            cap = 1000,
            fullOp = 2
        }
    }
}
```

- `topic`: 过滤消息并保存到队列的MQTT主题。
- `name`: MQ名称。
- `ringbus`: 配置环形队列的参数。

### ringbus

`ringbus` 用于存放MQTT消息，其配置参数包括：

- `name`: 环形队列名称。
- `cap`: 环形队列的容量大小（消息数量）。
- `fullOp`: 当环形队列满时的行为。提供以下四种选项：
  ```
  0: RB_FULL_NONE - 当环形队列满时，不执行额外动作，入队失败。
  1: RB_FULL_DROP - 当环形队列满时，将该消息丢弃，清空所有在环形队列中的数据并执行入队动作。
  2: RB_FULL_RETURN - 当环形队列满时，环形队列中的数据会放入aio进行返回。
  3: RB_FULL_FILE（依赖parquet） - 当环形队列满时，环形队列中的数据会写入文件中进行持久化存储。
  ```

注意：如果 `fullOp=3`或者`fullOp=2`，则需要开启 `PARQUET` 编译选项，并在 `nanomq.conf` 中配置 `parquet`。

## PARQUET 编译选项

要开启 `PARQUET` 编译选项，请执行以下命令：

```sh
cmake -DENABLE_PARQUET=ON ../
```

## 总结

通过配置 `exchange_client`、`exchange` 和 `ringbus`，可以实现对MQTT消息的过滤和存储。根据需求，可以设置环形队列的容量及满时的处理行为。如果选择 `fullOp=3`或者`fullOp=2`，需要额外配置 `PARQUET` 以实现持久化存储。