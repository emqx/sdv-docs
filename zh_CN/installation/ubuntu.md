# 基于 tar 包的安装部署

在本文中，我们将指导您如何在 linux 中完成 sdv-flow 及其所需组件的安装和使用。

## 安装条件

sdv-flow 安装前，请确认您的环境满足以下要求：

| OS             | 版本要求       |
| :------------- | :-------------|
| Ubuntu         | 18.04 或以上   |
| CentOS         | 8.0 或以上     |
| Debian         | 10 或以上      |

## 获取安装包

欢迎访问 EMQ 官网获取 sdv-flow 的安装包。

1. 进入[联系我们](https://www.emqx.com/zh/contact?product=emqx-ecp)页面。
2. 输入必要的联系信息，如姓名、公司、工作邮箱，国家和地区，以及您的联系方式。
3. 您可在下方的文本框中填写您的应用场景及需求，以便我们为您提供更好的服务。
4. 填写好以上信息后，点击 **立即提交**，我们的销售将会尽快与您联系。

## 安装 SDV Flow

将下载的安装包 `sdv-flow-*-linux.tar.gz` 上传到车机设备上，解压缩后，进入解压缩后的目录，得到文件列表如下。下面的被标注出来的几部分是我们需要关注的目录。

```bash
.
├── bin
│   └── sdv-flow # sdv-flow 可执行程序
├── core
├── data
│   ├── ekuiper
│   │   ├── data
│   │   └── plugins
│   ├── nanomq
│   └── sdv-flow
│       └── support_metric.csv
├── etc
│   └── sdv-flow.yaml # sdv-flow 配置文件
├── log # sdv-flow 默认生成的配置文件
└── software
    ├── ekuiper # kuiper 工作目录
    │   ├── bin
    │   ├── data
    │   ├── etc
    │   ├── log
    │   └── plugins
    └── nanomq # nanomq 工作目录
        ├── etc
        ├── log
        ├── nanomq
        ├── parquet
        └── readme.txt
```

首先需要修改数据接入配置文件 `software/nanomq/etc/nanomq.conf`， 将 `bridges.mqtt.emqx1` 的 server 地址修改为执行 sdv-platform 所配置的云端 emqx 的地址。
```conf
bridges.mqtt.emqx1 {
        server = "mqtt-tcp://broker.emqx.io:1883"
        proto_ver = 4
        keepalive = 60s
        backoff_max = 60s
        clean_start = false
        username = admin
        password = public
        ......
}
```

启动 sdv-flow
```bash
./bin/sdv-flow run # 后台启动可以执行 ./bin/sdv-flow start

```
可在 log 目录下的 sdv-flow.log 文件发现有日志生成，查看日志。可以看到 edgeagent registered successfully 的日志说明，服务启动正常并且已注册到 sdv-platform。
```bash
$ ./bin/sdv-flow run
time="2024-06-27T16:16:52+08:00" level=info msg="[Agent] is starting, vin: ubuntu" file="agent/init.go:18" func=sdv-flow/agent.Init
time="2024-06-27T16:16:52+08:00" level=error msg="path not exist" file="agent/clean_parquet.go:17" func=sdv-flow/agent.CleanParquetFile
time="2024-06-27T16:16:52+08:00" level=info msg="[MQTT]connected to tcp://127.0.0.1:1883" file="agent/mqtt.go:70" func=sdv-flow/agent.onMQTTConnected
time="2024-06-27T16:16:52+08:00" level=info msg="[Agent]Agent start registration to default org and project. " file="agent/init.go:47" func=sdv-flow/agent.Init
time="2024-06-27T16:16:52+08:00" level=info msg="[Agent]edgeagent registered successfully ,orgId : , projectId :" file="agent/registration.go:127" func=sdv-flow/agent.register.func1
time="2024-06-27T16:16:52+08:00" level=info msg="[Agent]heartbeat is enabled ,interval 15" file="agent/init.go:73" func=sdv-flow/agent.Init
time="2024-06-27T16:16:52+08:00" level=info msg="MQTT Tunnel Proxy Agent connected to tcp://127.0.0.1:1883" file="mqtt/proxy.go:88" func=ecp-tunnel/mqtt.SubTunnelTopic
time="2024-06-27T16:16:52+08:00" level=info msg="MQTT Tunnel Proxy Subscribed topic agent/ubuntu/proxy/request/+" file="mqtt/proxy.go:116" func=ecp-tunnel/mqtt.SubTunnelTopic.func1
```

## 验证安装
sdv-flow 和 sdv-platform 之间通过 MQTT 协议进行通信，其中包括，
- 车端通过 ecp 开头的 MQTT 主题上报状态
- 云端通过 agent 开头的 MQTT 主题下发规则或指令
因此 sdv-flow 启动并注册成功后，会不断向云端定时发送心跳信息，包括各个服务的运行状态，资源占用情况和指标信息，云端可以通过订阅主题 ecp/# 获得车端上报的信息（以下截图为 MQTT 客户端软件  MQTTX 订阅 ecp/# 的结果）。
同时可以订阅 `ecp/#` 获取边缘端状态，如图所示：![](_assets/sdv-flow-heartbeat.png)
