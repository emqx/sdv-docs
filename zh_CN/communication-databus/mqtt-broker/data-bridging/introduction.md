# MQTT 数据桥接

桥接是一种连接多个 MQTT 消息中间件的策略，其特性与集群模式显著不同。在桥接模式下，节点之间不进行主题树或路由表的复制操作。桥接模式的核心职能包括：

- 根据预定的规则，将消息转发至指定的桥接节点；
- 对桥接节点上的特定主题进行订阅，并在接收到消息后在本地节点或集群内进行传递和转发。

作为一种高效的网络协议，桥接模式极大地提高了 MQTT 消息传递的灵活性和可扩展性，因此在物联网通信中扮发挥着重要的作用。NanoMQ 现支持创建 MQTT over TCP/TLS 桥接和 MQTT over QUIC 桥接，进一步增强了跨网络通信的便捷性和效率。

## [MQTT over TCP/TLS 桥接](./tcp-tls-bridge.md)
本节将介绍 MQTT over TCP/TLS 数据桥接相关的配置参数，并将包含一个典型的 `nanomq.conf` 文件配置。本节还将介绍如何通过指定的配置文件运行 NanoMQ 以及如何对桥接进行测试。

## [MQTT over QUIC 桥接](./quic-bridge.md)
针对较难集成 MQTT over TCP 数据桥接的场景，NanoMQ 创新性地引入了 MQTT over QUIC 数据桥接。QUIC 最初由 Google 开发，后来被互联网工程任务组（IETF）采纳为全球标准。它是一种新的传输协议，提供更快的连接建立速度。通过 MQTT over QUIC 数据桥接，我们可以充分发挥 QUIC 协议在 IoT 场景中的优势。