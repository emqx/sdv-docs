# 文件传输功能

## 概述

文件传输功能集成在 消息总线 中作为一个服务，用于接收文件传输请求并将文件通过 MQTT 协议发送到本地 MQTT 消息服务器 的 TCP-MQTT 监听端口。如果需要将文件发送到云端，可以在 MQTT 消息服务器 的桥接选项中配置需要转发的主题。

## 1. 功能介绍

文件传输客户端服务用于接收文件传输请求，并将文件传输到指定的目标主题。它支持两种模式：通过 MQTT 订阅模式或使用 Push-Pull 模式。

## 2. 配置文件示例

```hocon
filetransfer {
    enable = true
    # 支持两种模式来接收文件传输请求
    # 1. MQTT 订阅模式
    # bind = "mqtt-tcp//127.0.0.1:1883"
    # 2. pushpull
    # bind = "IPC:///tmp/file_transfer"
    bind = "mqtt-tcp//127.0.0.1:1883"
    # MQTT 订阅模式下用于文件传输的主题
    topic = "file_transfer"
}
```

### 配置项说明

- **enable**: 控制文件传输功能是否开启。
- **bind**: 文件传输客户端接收文件传输命令的方式。
  - `mqtt-tcp://`: 通过 MQTT 订阅目标 URL 的主题，当有 MQTT 客户端向该 URL 的主题发布文件传输请求时，文件传输客户端可以接收并处理该请求。
  - `IPC://`: 文件传输客户端监听的 IPC 文件路径。
- **topic**: MQTT 订阅模式下用于接收文件传输请求的主题。

## 3. 文件传输请求格式

文件传输请求的格式如下：

```json
{
  "files": ["/tmp/1.log","/tmp/2.log"],
  "filenames": ["1.log","2.log"],
  "topics": ["topic/1","topic/2"],
  "delete": [-1,20],
  "request-id": ["xxxx","xxxx"],
  "echo-id":"xxxxxx"
}
```

### 参数说明

- **files**: 待传输文件的完整路径数组。
- **filenames**: 待传输文件的文件名数组。
- **topics**: 文件传输目标主题数组。
- **delete**: 文件传输后的删除选项数组，单位为秒。
  - 默认值为 `-1`，表示不删除文件。
  - 小于 `0` 时，文件传输成功后不删除文件。
  - 等于 `0` 时，文件传输成功后立即删除文件。
  - 大于 `0` 时，文件传输成功后在指定秒数后删除文件（最大值为 7 天）。
- **request-id**: 可选参数，用于返回文件传输结果。
- **echo-id**: 可选参数，用于在响应中返回给请求者的标识。

### 示例说明

- 文件 `1.log` 将会被发送到主题 `topic/1`，发送后 **不删除** 文件。
- 文件 `2.log` 将会被发送到主题 `topic/2`，发送后 **删除** 文件。

## 4. 文件传输结果

文件传输结果会发布到本地 MQTT 消息服务器 的 1883 端口，主题为 `file_transfer/result`。如果需要知道文件传输结果，可以订阅该主题的消息。

### 结果消息格式

```json
{
  "request-id": ["xxxx","xxxx"],
  "message": ["0","11"],
  "result": "success",
  "echo-id": "xxxxx"
}
```

示例说明：
- **request-id**: 对应文件传输请求中的 `request-id`。
- **message**: 错误码，`0` 代表成功，其他错误码见文档说明。
- **result**: 文件传输是否成功，`success` 表示成功，`fail` 表示失败。
- **echo-id**: 对应文件传输请求中的 `echo-id`。

## 5. 文件传输功能例子

当 sdv-flow 启动后，首先检查 MQTT 消息服务器 是否正常运行。然后文件传输客户端服务可以通过以下两种方式工作：

- 通过订阅 MQTT 消息服务器 的 `file_transfer` 主题来接收文件传输请求：

```hocon
filetransfer {
    enable = true
    bind = "mqtt-tcp//127.0.0.1:1883"
    topic = "file_transfer"
}
```

- 监听 IPC 文件 `/tmp/file_transfer` 来接收文件传输请求：

```hocon
filetransfer {
    enable = true
    bind = "IPC:///tmp/file_transfer"
}
```

## 注意事项

- 配置文件中的 `bind` 和 `topic` 需要根据具体情况进行调整。
- 确保目标主题的正确性和可达性，以及文件传输后的文件处理方式。