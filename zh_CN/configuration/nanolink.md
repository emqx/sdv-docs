# 多协议网关 NanoLink 协议转换配置
## 配置说明

NanoLink 的默认配置文件格式是 HOCON。HOCON（Human-Optimized Config Object Notation）是 JSON 的超集，非常适合存储易于人类读写的配置数据。你可以在 `software/nanolink/etc/` 目录找到配置文件 `nanolink.conf`。包括 MQTT、AVTP、UDP、SOME/IP 等部分的配置。以下内容基于 HOCON 配置文件格式。

## 配置文件语法

在配置文件中，值可以用类似 JSON 的对象来表示，例如：

```bash
log {
    dir = "/var/log"
    file = "nanolink.log"
}
```

另一种等价的表示方法是扁平格式，例如：

```bash
log.dir = "/var/log"
log.file = "nanolink.log"
```

这种扁平格式几乎与 NanoLink 的配置文件格式向后兼容（即所谓的 'cuttlefish' 格式）。

但这并不完全兼容，因为 HOCON 经常要求字符串两端加上引号。例如：

- cuttlefish: `mqtt.bind = 0.0.0.0:1883`
- HOCON: `mqtt.bind = "0.0.0.0:1883"`

### 配置重载规则

HOCON 的值是分层覆盖的，普遍规则如下：

- 在同一个文件中，后定义的值（在文件底部）会覆盖先定义的值（在文件顶部）。
- 当按层级覆盖时，高层级的值会覆盖低层级的值。

例如，在如下配置中，最后一行的 `level` 值会覆盖原先的 `error` 值，而 `to` 字段保持不变：

```bash
log {
    to = [file, console]
    level = error
}

log.level = debug
```

## 参数说明

### nanolink.mqtt

#### 基本配置参数

| 参数名                                | 数据类型    | 参数说明                                    |
| ------------------------------------- | ----------- | ------------------------------------------- |
| address                               | String      | 指定 MQTT 远程桥接服务器的地址，格式为 `host:port`。 |
| clientid                              | String      | 用于标识 MQTT 远程桥接客户端的 `ClientId`。 |
| keepalive                             | Duration    | 指定 MQTT `keepalive` 的时间间隔，单位为秒。 |
| clean_start                           | Boolean     | 指定 `Clean Start` 标志，决定是否清除上次会话的状态。 |
| username                              | String      | 指定用于远程桥接的 MQTT 服务器的用户名。     |
| password                              | String      | 指定用于远程桥接的 MQTT 服务器的密码。       |
| forwards.avtp_to_mqtt.topic           | String      | 指定收到的 AVTP 消息要转发到 MQTT broker 的主题。     |
| forwards.avtp_to_mqtt.qos             | Integer     | 指定收到的 AVTP 消息转发时的 QoS（服务质量）级别。      |
| forwards.udp_to_mqtt.topic            | String      | 指定收到的 UDP 要转发到 IoTHub 的主题。              |
| forwards.udp_to_mqtt.qos              | Integer     | 指定收到的 UDP 消息转发时的 QoS（服务质量）级别。      |

### nanolink.avtp

| 参数名                                | 数据类型    | 参数说明                                    |
| ------------------------------------- | ----------- | ------------------------------------------- |
| enable                                | Boolean     | 启用或禁用 AVTP 功能。                       |
| ifname                                | String      | 指定用于 AVTP 通信的网络接口名称。           |
| macaddr                               | String      | 指定 AVTP 流的目标 MAC 地址。                |

### nanolink.udp

| 参数名                                | 数据类型    | 参数说明                                    |
| ------------------------------------- | ----------- | ------------------------------------------- |
| enable                                | Boolean     | 启用或禁用 UDP 功能。                        |
| addr                                  | String      | 指定 UDP 监听的 IP 地址。                   |
| port                                  | Integer     | 指定 UDP 监听的端口号。                     |

### nanolink.someip

| 参数名                                | 数据类型    | 参数说明                                    |
| ------------------------------------- | ----------- | ------------------------------------------- |
| enable                                | Boolean     | 启用或禁用 SOME/IP 功能。                    |
| prefix                                | String      | 指定 MQTT 主题前缀。                        |
| commonapi_config                      | String      | 指定 CommonAPI 配置文件的路径。              |

### nanolink.log

| 参数名                                | 数据类型    | 参数说明                                    |
| ------------------------------------- | ----------- | ------------------------------------------- |
| enable                                | Boolean     | 启用或禁用日志记录功能。                     |
| level                                 | String      | 指定日志记录的详细程度。                     |
| dir                                   | String      | 指定日志文件存储路径。                       |
| file                                  | String      | 指定日志文件名。                             |