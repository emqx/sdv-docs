# 快速开始

随着车联网与 5G 技术的融合以及车辆智能化的发展，车辆产生的数据越来越多，数据驱动和场景闭环的重要性也凸显出来。智能驾驶数据闭环是智能驾驶技术发展中的一个重要概念，它指的是通过收集、处理和分析自动驾驶汽车在实际道路环境中产生的数据，来不断优化和升级智能驾驶算法和系统的过程。这个闭环系统通常包括以下几个关键环节：

- 数据采集
- 数据标注与清洗
- 模型训练、部署等

sdv-flow 可以通过编写基于类 SQL 的数据处理规则，并通过 API 热创建、热更新（无需 OTA) 到车端（量产车）运行，实现数据闭环。如何获取从大规模数据中获取有用的数据，获取之后如何有效回传、标注以及训练，是数据闭环的核心任务，也是大多数车企迫切想要拥有的能力。本教程将介绍如何使用 EMQ 提供的产品系列来实现上述功能。

## 软件安装

系统的部署架构示意图如下，本次软件安装主要是在车端的 sdv-flow, 以及在云端部署的 sdv-platform。图中的 Applications 是用户基于 sdv-platform 提供的 API 和大数据系统 Kafka 等开发的应用。

![](_assets/deployment-architecture.png)

建议读者使用 Ubuntu 操作系统，在开始安装之前，请确认以下前置条件：

- 准备一台机器用于安装 sdv-flow, Ubuntu 18.04 或以上
  - 建议机器配置 2c/2G，AMD64 或者 ARM64 CPU
- 准备一台机器用于安装 sdv-platform， 环境需要满足 
  - Ubuntu 20.04 或 22.04 
  - CentOS 7.0 或以上
  - 安装 [Docker](https://www.docker.com/) 20.10.0 或以上，安装成功并已启动。
  - 安装 [Docker Compose](https://docs.docker.com/compose/install/) 1.27.1 或以上
  - 建议 16c/32G，AMD64 机器
- 上述两台机器之间可以进行网络通信

### sdv-platform（云端）

在云端安装的软件列表如下，建议读者使用 docker-compose 进行安装。

| **软件名称**     | **版本** | **描述**                                                     |
| ---------------- | -------- | ------------------------------------------------------------ |
| sdv-platform     | latest   | 云端管理平台，可提供车辆注册，管理，云边通道，监控等服务。   |
| EMQX Enterprise  | latest   | [EMQX 企业版](https://docs.emqx.com/zh/enterprise/latest/)   |
| redis            | 6.2.6    | 开源的高性能键值对数据库，用于存储心跳等信息。               |
| mysql            | 8.0.18   | 开源的关系型数据库，用于存储需要持久化的数据。               |
| prom/prometheus  | v2.37.9  | 开源的系统监控和告警工具，车端监控数据最终推送到prometheus中，可提供灵活的查询语言和多种可视化功能，用于监控车端的性能和健康状况。 |
| prom/pushgateway | v1.6.2   | Pushgateway 是 Prometheus 的一个组件，它采用被动推送的方式将监控数据收集到 Prometheus 服务器中。 |

- 安装包解压

  ```shell
  tar zxvf emqx-sdv-platform-docker-compose-installer-*.tar.gz
  ```

- 解压得到 docker-compose 目录，包含如下文件

  ```
  .
  ├── configs
  ├── datavolumes
  ├── docker-compose.yaml
  ├── emqx_sdv_platform_ctl
  ├── prometheus
  ├── README.md
  ├── sdv_platform.conf
  └── templates
  ```

- 执行 `./emqx_sdv_platform_ctl --help` 获取帮助信息

- 执行 `./emqx_sdv_platform_ctl configure` 进行配置，只需要指定安装 sdv-platform 镜像版本及安装数据文件夹，或直接选择默认即可

  ```
  $  ./emqx_sdv_platform_ctl configure
  Generating docker-compose .env file
  Please input EMQX SDV PLATFORM image tag (default: latest): 
  Please input EMQX SDV PLATFORM data volume path (default: ./datavolumes/): 
  copy prometheus.yaml files ...
  Generating docker-compose env file ...
  Generating SDV PLATFORM config files ...
  Generating ecp.yaml ...
  All configurations are done.
  ```

- 执行 `./emqx_sdv_platform_ctl start` 启动 compose 容器，结果如下，

  ```
  $ ./emqx_sdv_platform_ctl start
  [+] Running 7/7
   ⠿ Network docker-compose_default  Created                                                                                                                      0.1s
   ⠿ Container sdv-pushgateway       Started                                                                                                                      3.2s
   ⠿ Container sdv-db                Started                                                                                                                      3.2s
   ⠿ Container sdv-prometheus        Started                                                                                                                      3.3s
   ⠿ Container sdv-redis             Started                                                                                                                      3.4s
   ⠿ Container sdv-emqx              Started                                                                                                                      3.3s
   ⠿ Container sdv-platform          Started 
  ```

- 执行 `./emqx_sdv_platform_ctl init` 创建管理员以及默认的组织和项目，结果如下，

  ```
  $ ./emqx_sdv_platform_ctl init  
  Please input email: admin@emqx.io
  Please input password: 
  Please input password again: 
  Creating admin admin@emqx.io ...
  Getting access token ...
  {"tokenType":"Bearer","accessToken":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJFTVFYIEVDUCIsInN1YiI6IjhlNThmN2I4IiwiZXhwIjoxNzIzMTAxMDU1LCJuYmYiOjE3MjMwODY2NTUsImlhdCI6MTcyMzA4NjY1NSwianRpIjoiNGFmYTEzN2UtZjljYy00Y2Q4LWEwMDctODc5M2JhMzA4OTFlIiwidHlwIjoiQmVhcmVyIn0.SljCPXR6IsJ56p1RMhYaFzfC0E7VorRr7e5BmZ5SXerP0C__hfIi6QhbW2Dnju2fKD_IzD31_sYIjQ07mxck4o11SA8d4_YsG-icJ1ktkQ3nkdmXXuuKSKdApD2k24DETfxXA4cpE8-oG039IAZtufLBPfwQkLL1SvHPcNj6Da7VWa-VsG1b9S5abpDnssAXdWOtHRdCWOwYUpkPSkK55wV0OF0rEJOyYaJP9rOkOUkJzIvMb8Lj66etAxiLHfEfL4o2y6HL3rR7O-xqIAvX9gG0udXSeXEfDUwUxKacA7L0EWHGzUVYFVba6ADHdJ_Al7IGPA1eV7PDBFYE0fpEEw","expiresIn":14400,"refreshToken":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJFTVFYIEVDUCIsInN1YiI6IjhlNThmN2I4IiwiZXhwIjoxNzIzMTczMDU1LCJuYmYiOjE3MjMwODY2NTUsImlhdCI6MTcyMzA4NjY1NSwianRpIjoiM2NhZjc2MzQtZTdlYS00YjgyLTgxNjUtZmU4NDRmMDRjNTkxIiwidHlwIjoiUmVmcmVzaCJ9.f_j86gWOLFLS-zUm6-a3-ovJmIhYxX20Ko1G7WbJMwfKl95OtPSAO7fV6ETYgRaE4oNNLCc1x2F_hQFC5IolC_91dwbAPZW4YYK1wcjkwr0l3fDvXMNOTcQEqMA8THzAbylB2lce8RqD2264YWBzt8kMhLfJTbTzqzskQWN-wm6_ftjHCS-AgLxk5oJK7ZOH2hasPp2CHmD2tKX2EyOWcWnDoBF-wNFOG_N-Kas1-thYeXJqxg6BiPXfUaDfyxLGLDjj1lmCqgesvZYXz_CYjdumg0g--5k9vHstFX-9QfLKEci2UFQdZ0TttKkzhDPp_0XG1i7v6JJ4yyE6O-OA8Q","refreshExpiresIn":86400,"admin":true}
  {"userId":"8e58f7b8","name":"admin","email":"admin@emqx.io","mobile":"","admin":true}
  Creating org default-org ...
  {"id":"67fe214f","name":"default-org","members":[{"userId":"8e58f7b8","roleIds":[1,0]}],"createdAt":"2024-08-08T03:10:55.908982278Z"}
  {"data":[{"id":"67fe214f","name":"default-org","projectCount":0}]}
  Creating project default-project ...
  {"id":"a3196d0d","name":"default-project","members":[{"userId":"8e58f7b8","roleIds":[1]}],"createdAt":"2024-08-08T03:10:56.074284435Z"}
  
  ```

- 查看容器是否全部启动，如下所示

  ```
  $ ./emqx_sdv_platform_ctl status
  NAME                COMMAND                  SERVICE             STATUS              PORTS
  sdv-db              "docker-entrypoint.s…"   db                  running             33060/tcp, 0.0.0.0:3307->3306/tcp
  sdv-emqx            "/usr/bin/docker-ent…"   emqx                running (healthy)   4370/tcp, 0.0.0.0:1883->1883/tcp, 0.0.0.0:8083-8084->8083-8084/tcp, 0.0.0.0:8883->8883/tcp, 0.0.0.0:18083->18083/tcp, 5369/tcp
  sdv-platform        "/ecp/sdv-platform"      sdv-platform        running             0.0.0.0:8082->8082/tcp
  sdv-prometheus      "/bin/prometheus --c…"   prometheus          running             0.0.0.0:31900->9090/tcp
  sdv-pushgateway     "/bin/pushgateway"       pushgateway         running             0.0.0.0:31901->9091/tcp
  sdv-redis           "docker-entrypoint.s…"   redis               running             0.0.0.0:6379->6379/tcp
  ```

### sdv-flow（车端）

- 下载安装包

- 安装包解压，下面以二进制压缩包为例：

```shell
tar zxvf sdv-flow-*-arm64-linux.tar.gz
```

解压后得到文件目录如下：

```
.
├── bin # sdv-flow 可执行程序
├── core
├── data
├── etc # sdv-flow 配置文件 sdv-flow.yaml 
├── log # sdv-flow 默认生成的 log 文件
└── software
```

运行 sdv-flow

```shell
./bin/sdv-flow run 
```

> 如果想在后台运行，可以使用命令 ：  `./bin/sdv-flow start`

接下来通过 HTTP API 修改 sdv-flow 连接云端的地址，将 `bridges.mqtt.emqx1` 的 server 地址修改为上一步中所安装的 emqx 地址 `{emqx_addr`}。如下：

```shell
$ curl -i --basic -u admin:public -X PUT 'http://localhost:14260/api/nanomq/bridges/emqx1' -d '{
"emqx1": {
    "name": "emqx1",
    "enable": true,
    "parallel": 2,
    "server": "mqtt-tcp://broker.emqx.io:1883",
    "proto_ver": 4,
    "clean_start": false,
    "username": "admin",
    "password": "public",
    "keepalive": 60,
    "forwards": [
      {
        "remote_topic": "",
        "local_topic": "ecp/#"
      }
    ],
    "subscription": [
      {
        "remote_topic": "agent/#",
        "local_topic": "",
        "qos": 1
      }
    ]
  }
}'
```

如果成功，上述的 curl 命令会收到如下返回：

```json
{"code":0}
```

可在 log 目录下的 sdv-flow.log 文件发现有日志生成，查看日志。可以看到 `edgeagent registered successfully` 的日志说明，服务启动正常并且已注册到 sdv-platform。

```
time="2024-06-27T16:16:52+08:00" level=info msg="[Agent] is starting, vin: ubuntu" file="agent/init.go:18" func=sdv-flow/agent.Init
time="2024-06-27T16:16:52+08:00" level=error msg="path not exist" file="agent/clean_parquet.go:17" func=sdv-flow/agent.CleanParquetFile
time="2024-06-27T16:16:52+08:00" level=info msg="[MQTT]connected to tcp://127.0.0.1:1883" file="agent/mqtt.go:70" func=sdv-flow/agent.onMQTTConnected
time="2024-06-27T16:16:52+08:00" level=info msg="[Agent]Agent start registration to default org and project. " file="agent/init.go:47" func=sdv-flow/agent.Init
time="2024-06-27T16:16:52+08:00" level=info msg="[Agent]edgeagent registered successfully ,orgId : , projectId :" file="agent/registration.go:127" func=sdv-flow/agent.register.func1
time="2024-06-27T16:16:52+08:00" level=info msg="[Agent]heartbeat is enabled ,interval 15" file="agent/init.go:73" func=sdv-flow/agent.Init
time="2024-06-27T16:16:52+08:00" level=info msg="MQTT Tunnel Proxy Agent connected to tcp://127.0.0.1:1883" file="mqtt/proxy.go:88" func=ecp-tunnel/mqtt.SubTunnelTopic
time="2024-06-27T16:16:52+08:00" level=info msg="MQTT Tunnel Proxy Subscribed topic agent/ubuntu/proxy/request/+" file="mqtt/proxy.go:116" func=ecp-tunnel/mqtt.SubTunnelTopic.func1
```

### 验证安装

sdv-flow 和 sdv-platform 之间通过 MQTT 协议进行通信，其中包括：

- 车端通过 `ecp` 开头的 MQTT 主题上报状态
- 云端通过 `agent` 开头的 MQTT 主题下发规则或指令

sdv-flow 启动后，将自动获得 vin 码，代表车端的唯一标识。默认为所在机器的 hostname (vin码的生成可以定制)。 sdv-flow 注册成功后，会不断向云端定时发送心跳信息，包括各个服务的运行状态，资源占用情况和指标信息，云端可以通过订阅主题 `ecp/#` 获得车端上报的信息（以下截图为 MQTT 客户端软件  [MQTTX ](https://docs.emqx.com/zh/enterprise/latest/messaging/publish-and-subscribe.html#mqttx)订阅 `ecp/#` 的结果）。

![heatbeat-mqttx](_assets/heatbeat-mqttx.png)

## 接入车载数据并编写数据处理规则

### 业务场景描述 

假设目前有个场景，车企需要研究用户转弯时的驾驶习惯，以提升个性化的辅助驾驶能力。首先，我们需要采集车辆转弯时的相关驾驶数据。全量采集数据固然包含全部所需的数据，然而数据数量太多浪费大量带宽，同时数据质量不佳，模型训练难以利用。利用 sdv-flow，**我们可以创建一个简单的规则，仅当车辆转弯时，采集相关联的驾驶数据信号，实时转换成模型所需格式，并通过 MQTT 实时传送到云端数据中心。后续需要采集更多信号时，我们直接通过 API 热更新规则，即可快速迭代。**

![](_assets/sdv-architecture.png)

1. 注册
2. 通过 HTTP API 创建规则
3. 规则创建请求通过 MQTT 转发给 sdv-flow
4. 规则创建结果返回给云端

### 注册

sdv-flow 通过前面安装过程中 `./bin/sdv-flow start` 命令启动后，会自动向云端 sdv-platform 发起注册请求，在云端 sdv-platform 软件授权足够的情况下，注册成功后 sdv-flow 会定时向云端发送心跳消息。

### 规则创建与运行

在实验场景中，我们将采用 MQTT 模拟数据源，实际场景中需要定制接入车载数据源。实现转弯数据采集和清洗场景，我们只需要以下三个步骤：

1. 准备用于解析总线数据的 DBC 文件。
2. 创建数据流，接入 MQTT 模拟数据。
3. 创建针对该数据流的数据采集和处理规则。

其中，步骤 1 和 2 通常只需要初始化的时候执行一次。后续用户可以在同一个数据流上创建更多的数据处理规则。

#### DBC 文件准备

DBC 表示“CAN数据库”，它是一种用于描述 CAN（控制器局域网）通信中数据结构的文件格式。它提供了一种标准化的方式来定义 CAN 总线系统的消息、信号、属性和值。DBC 文件通常用于汽车行业的车辆网络通信，但也可以在其他实施 CAN 总线的应用中使用。为了将总线上的二进制数据解析为有意义的信号数据，用户需要准备对应的 DBC 文件，并部署到车端。这样 SDV platform 才能在车端对数据进行解析。

**注意**：DBC 文件可以只包含车端需要用到的信号定义。解析时，未在 DBC 中定义的 CAN 信号将会忽略。

实验中，我们将使用附件中的 demo.dbc 文件。请将文件部署到车端目录，例如 `/opt/dbc` 中。创建数据流时，我们需要指定 DBC 目录所在的位置。

**鉴权准备**

登录，获取auth token

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/account/login' \
--header 'Content-Type: application/json' \
--data-raw '{"username":"user","password":"pwd"}'
```

结果示例：

```json
{
    "tokenType": "Bearer",
    "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJFTVFYIEVDUCIsInN1YiI6IjEzNmQ4MTcyIiwiZXhwIjoxNzE5NTY1OTQ5LCJuYmYiOjE3MTk1NTE1NDksImlhdCI6MTcxOTU1MTU0OSwianRpIjoiYzZkNTljNGYtOTY0YS00ZGY2LWJjZjItZDc0NmUwNmFhZDhjIiwidHlwIjoiQmVhcmVyIn0.cl4x9CfZAfLWNgOXRvqK6j3eGTGtyn0Yl6GGKxhf56ehBjyIXWXkC5Vd8I1neOzYjSXpW1T3fkIX2PEi5FkSVGXZpFHlTlJ_H8ULFsi472Ww-CJIcqs7l4AFzM2soJ8WZddJW3BMlL0aMckRBaGF9r_IkEJwqJwxafeJwqhop24ErEsv0FMHhPVprB7bVeyPeAnS6K0e0KheX_p2QFMNOWGtEyA90Jblqb7iusr3QrrErDPZIEbIweMlban1Kr4B9BqVo8QvJZneFOF3VNuJprcelK9Q5O79yh-7HlaFAlTVBDFeMQiw4OZdoGxYUKzFSD8WClwCcaWyfJlL17pfvQ",
    "expiresIn": 14400,
    "refreshToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJFTVFYIEVDUCIsInN1YiI6IjEzNmQ4MTcyIiwiZXhwIjoxNzE5NjM3OTQ5LCJuYmYiOjE3MTk1NTE1NDksImlhdCI6MTcxOTU1MTU0OSwianRpIjoiNTQ0ZmZjNDItYjEzYS00ZGZmLTllZWMtZDAwYWE0MGRhNDhhIiwidHlwIjoiUmVmcmVzaCJ9.W4ftwjn5cJEPolowwQX2_zRDIDZkHijHsKdUGogzuefsBVqLdSDeGJKD3wGhhlaqzWjgI2FT3VlhO_6PL4gLN7H3EeDPMtQsvXV7fVocLZ557ULp5gc7vFUZNuFzNY7GZBhhhK187YOHM4WtiRBq-tfumaT7xnb5Auwd9TishDDLkQaY4BM8uEX5JxXKH-lCXlXEbBrD6YxytHXBPZymzH_tA5vLPyRsKszMoNjBqYhf4Pjp3RI9UgLMfy5o6ue5vUL7Zpx1Qa_H8s7w7n4a9hHTq6b4uz7cHAHULVvj8fUD4zcPQdVFQIjB0P1EzVGq4ltpGkWbOaAmRciXWih4Tg",
    "refreshExpiresIn": 86400,
    "admin": true
}
```

其中 accessToken 即为 auth token，后续的请求，在请求头中都加入Authorization: Bearer {auth token}

#### 数据流

在 sdv-platform 中，通过 REST API 下发数据流创建命令。`${vin}`代表车端的vin码，需自行替换。

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/edgeservice/proxy/${vin}/streams' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer ${auth token}' \
--data-raw '{
    "sql": "CREATE STREAM canDemo() WITH (TYPE=\"mqtt\",FORMAT=\"canjson\",DATASOURCE=\"canudp\",SHARED=\"true\",SCHEMAID=\"/opt\/dbc\");"
}
```

该指令定义了一个名为 `canDemo` 的数据流。其中包括参数：

- TYPE="mqtt"： 表示数据流类型为 MQTT，将通过 MQTT 协议接入数据。
- DATASOURCE="canudp"：数据源为 `canudp`，对于 MQTT 协议来说，数据源即订阅的主题。
- FORMAT="canjson"：数据格式为 `canjson`，该格式为 SDV flow 定义的基于 JSON 的数据格式，具体格式将在数据章节将详细描述。请注意，该格式仅用于演示。实际项目中，用户通常会使用更高效的数据格式
- SCHEMAID="/opt/dbc"：对于 canjson 格式而言，该选项用于指定 DBC 文件的目录。此处设置为上一步骤所部署的 DBC 文件的目录。
- SHARED="true"：表示数据流为可共享的。多个规则同时使用该数据流时，处理是同一份数据。

作为实验，我们直接使用 MQTT 协议发送包含 CAN 帧的测试数据。在实际应用中，车企通常有专门的总线数据透传应用，sdv-flow 可对接各种协议和格式。

我们的模拟数据使用的 DBC 文件如下，其中 DRIVE 消息中包含简化的驾驶数据：

- Angle：转弯角度
- Speed: 车速
- Acceleration：加速度

我们的实验规则将使用这部分的信号作为示例。

```
BO_ 423 GPS: 8 Vector__XXX
	SG_ Latitude : 7|15@0+ (0.01,-90) [-90|90] "" Vector__XXX
	SG_ Longtitude : 23|16@0+ (0.01,-180) [-180|180] "" Vector__XXX
	SG_ Altitude : 39|20@0+ (0.01,-1000) [-1000|9000] "" Vector__XXX
BO_ 485 DRIVE: 8 Vector__XXX
	SG_ Angle : 7|8@0+ (1,0) [0|180] "" Vector__XXX
	SG_ Speed : 15|14@0+ (0.01,0) [0|120] "" Vector__XXX
	SG_ Acceleration : 17|8@0+ (0.01,-1) [-1|1] "" Vector__XXX
BO_ 555 ENGINE: 8 Vector__XXX
	SG_ Temperature : 21|9@0+ (1,-20) [-20|300] "" Vector__XXX
	SG_ RPM : 39|13@0+ (1,0) [0|6000] "" Vector__XXX
	SG_ Torque : 55|7@0+ (1,0) [0|0] "" Vector__XXX
BO_ 423 PEDAL: 8 Vector__XXX
	SG_ AcceleratorOpeningAngel : 15|14@0+ (0.01,0) [0|100] "" Vector__XXX
	SG_ BrakePosition : 31|14@0+ (0.01,0) [0|100] "" Vector__XXX
BO_ 312 DASHBOARD: 8 Vector__XXX
	SG_ Odometer : 47|24@0+ (1,0) [0|0] "" Vector__XXX
	SG_ FuelConsumptionRate : 7|10@0+ (0.01,0) [0|0] "" Vector__XXX
	SG_ FuelUsed : 23|17@0+ (1,0) [0|0] "" Vector__XXX
```

#### 数据处理规则

##### 输入数据

从数据流中，我们会收到 CAN 总线原始数据。而在规则中，我们可以直接操作解析后的信号，并做相应的计算。模拟 JSON 数据格式如下所示：

```json
{
  "ts":1617180010000, 
  "frames":[
    {"id":485, "bus":5,"d":0,"t":1617179000000,"data":"546573740000AB00"},
    {"id":555,"bus":6,"d":0,"t":1617179000100, "data":"5465737400000000"}
  ]
}
```

一条 JSON 消息中，聚合了多个 CAN 帧，定义在 frames 中。每个 frame 的格式至少需要包括：

- id: CAN id 值。
- data: CAN payload 的 hex string。

数据流会根据配置的 DBC 进行解析，进入规则的数据为解析后的 KV 数据，例如：

```json
{
  "ts":1617180010000,
  "DRIVE": {
    "id":           485,
    "bus":          4,
    "d":            0,
    "t":            1617179000000,
    "Angle":        124,
    "Speed":        0,
    "Acceleration": 0
  },
  "ENGINE": {
    "id":          555,
    "bus":         4,
    "d":           0,
    "t":           1617179000100,
    "Temperature": 81,
    "RPM":         598,
    "Torque":      10
  }
}
```

在规则的 SQL 语句中，我们可以基于这个数据结构进行编写。例如，获取转弯角度可用 `SELECT DRIVE.angle`。

##### 编写规则

我们的规则是在车辆急转弯时，采集车辆的转弯角度，速度，加速度，发动机温度和转速，并做简单的数据转换（将嵌套的信号数据平铺）。规则配置如下：

```json
{
  "id": "ruleFetch",
  "name": "采集转弯数据",
  "sql": "SELECT DRIVE.Angle as angle, DRIVE.Speed as speed, DRIVE.Acceleration as acceleration, ENGINE.temperature as engineTemp, ENGINE.RPM as rpm FROM canDemo WHERE DRIVE.Angle > 10",
  "actions": [
    {
      "mqtt": {
        "server": "tcp://remote:1883",
        "topic": "result/angle",
        "sendSingle": true
      }
    }
  ]
}
```

其中，规则 id 为规则的不可重复的 id，后续可根据 id 进行更改删除等管理操作。name 为规则名字，可描述规则的用途等。 规则的业务逻辑通过 sql 定义；actions 则定义规则执行命中后的动作。

本规则的 SQL 语句中，主要有几个部分：

- FROM canDemo: 从上一节定义的数据流 canDemo 获取数据
- WHERE DRIVE.Angle > 10: 当角度大于 10 时触发。
- SELECT 部分：选择采集的信号，并重命名/平铺。

Actions 中定义了采集命中后，结果发送到 MQTT 的 `result/angle` 主题中。

##### 运行规则

通过 sdv-platform 下发规则创建指令。规则可自动运行。通过 sdv-platform 查看规则运行状态。

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/edgeservices/config' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer ${auth token}' \
--data-raw '{
    "ids": [
        "${vin}"
    ],
    "method": "POST",
    "path": "/rules",
    "body": "{\n  \"id\": \"ruleFetch\",\n  \"name\": \"采集转弯数据\",\n  \"sql\": \"SELECT DRIVE.Angle as angle, DRIVE.Speed as speed, DRIVE.Acceleration as acceleration, ENGINE.temperature as engineTemp, ENGINE.RPM as rpm FROM canDemo WHERE DRIVE.Angle > 10\",\n  \"actions\": [\n    {\n      \"mqtt\": {\n        \"server\": \"tcp://remote:1883\",\n        \"topic\": \"result/angle\",\n        \"sendSingle\": true\n      }\n    }\n  ]\n}\n"
}'
```

查看某个车端当前规则的运行状态

```shell
curl --location --request GET 'http://127.0.0.1:8082/api/edgeservice/proxy/${vin}/rules' \ 
--header 'Authorization: Bearer ${auth token}' \
```

## 数据处理

从车到云的数据处理流程如下图所示

![data-processing-flow](_assets/data-processing-flow.png)

1. Data bus 读取 CAN 数据，在本文中使用的是模拟的 MQTT 数据包，在实际案例中是对接车端的各类数据源
2. Data engine 处理数据，如果数据需要落磁盘则发送指令给 data bus 
3. 数据存储落盘
4. 将结果数据通过 MQTT 协议发送给云端
5. sdv-platform 将数据对接到大数据系统

### 发送模拟数据

根据 [数据流定义](http://localhost:63342/markdownPreview/699592024/markdown-preview-index-316530953.html?_ijt=73lidq25r5ri4lkqeguv4qc0e8#数据流)，我们的数据流监听的是 sdv-flow 中的 `canudp` MQTT 主题。在车端，我们可通过 MQTTX 发送测试数据：

```json
{
  "ts":1617180010000, 
  "frames":[
    {"id":485, "bus":5,"d":0,"t":1617179000000,"data":"546573740000AB00"},
    {"id":555,"bus":6,"d":0,"t":1617179000100, "data":"5465737400000000"}
  ]
}
```

该数据中，Angle 值为 84，应当能够触发采集。 通过 sdv-platform API 查看规则状态，如下所示。规则状态指标记录了规则的每个算子的进入和输出的消息数，其中 `source_canDemo_xxx` 为数据流；`op_3_filter_xxx` 为 where 语句生成的过滤算子。若不满足条件，则可发现其 records_in 比 records_out 多。

```json
{
  "status": "running",
  "lastStartTimestamp": "1718856550022",
  "lastStopTimestamp": "0",
  "nextStopTimestamp": "0",
  "source_canDemo_0_records_in_total": 1,
  "source_canDemo_0_records_out_total": 1,
  "source_canDemo_0_messages_processed_total": 1,
  "source_canDemo_0_process_latency_us": 0,
  "source_canDemo_0_buffer_length": 0,
  "source_canDemo_0_last_invocation": "2024-06-20T12:09:44.898669",
  "source_canDemo_0_exceptions_total": 0,
  "source_canDemo_0_last_exception": "",
  "source_canDemo_0_last_exception_time": 0,
  "op_canDemo_2_decoder_0_records_in_total": 1,
  "op_canDemo_2_decoder_0_records_out_total": 1,
  "op_canDemo_2_decoder_0_messages_processed_total": 1,
  "op_canDemo_2_decoder_0_process_latency_us": 260,
  "op_canDemo_2_decoder_0_buffer_length": 0,
  "op_canDemo_2_decoder_0_last_invocation": "2024-06-20T12:09:44.898669",
  "op_canDemo_2_decoder_0_exceptions_total": 0,
  "op_canDemo_2_decoder_0_last_exception": "",
  "op_canDemo_2_decoder_0_last_exception_time": 0,
  "op_3_filter_0_records_in_total": 1,
  "op_3_filter_0_records_out_total": 1,
  "op_3_filter_0_messages_processed_total": 1,
  "op_3_filter_0_process_latency_us": 0,
  "op_3_filter_0_buffer_length": 0,
  "op_3_filter_0_last_invocation": "2024-06-20T12:09:44.898929",
  "op_3_filter_0_exceptions_total": 0,
  "op_3_filter_0_last_exception": "",
  "op_3_filter_0_last_exception_time": 0,
  "op_4_project_0_records_in_total": 1,
  "op_4_project_0_records_out_total": 1,
  "op_4_project_0_messages_processed_total": 1,
  "op_4_project_0_process_latency_us": 71,
  "op_4_project_0_buffer_length": 0,
  "op_4_project_0_last_invocation": "2024-06-20T12:09:44.898929",
  "op_4_project_0_exceptions_total": 0,
  "op_4_project_0_last_exception": "",
  "op_4_project_0_last_exception_time": 0,
  "op_mqtt_0_0_transform_0_records_in_total": 1,
  "op_mqtt_0_0_transform_0_records_out_total": 1,
  "op_mqtt_0_0_transform_0_messages_processed_total": 1,
  "op_mqtt_0_0_transform_0_process_latency_us": 0,
  "op_mqtt_0_0_transform_0_buffer_length": 0,
  "op_mqtt_0_0_transform_0_last_invocation": "2024-06-20T12:09:44.899",
  "op_mqtt_0_0_transform_0_exceptions_total": 0,
  "op_mqtt_0_0_transform_0_last_exception": "",
  "op_mqtt_0_0_transform_0_last_exception_time": 0,
  "op_mqtt_0_1_encode_0_records_in_total": 1,
  "op_mqtt_0_1_encode_0_records_out_total": 1,
  "op_mqtt_0_1_encode_0_messages_processed_total": 1,
  "op_mqtt_0_1_encode_0_process_latency_us": 0,
  "op_mqtt_0_1_encode_0_buffer_length": 0,
  "op_mqtt_0_1_encode_0_last_invocation": "2024-06-20T12:09:44.899",
  "op_mqtt_0_1_encode_0_exceptions_total": 0,
  "op_mqtt_0_1_encode_0_last_exception": "",
  "op_mqtt_0_1_encode_0_last_exception_time": 0,
  "sink_mqtt_0_0_records_in_total": 1,
  "sink_mqtt_0_0_records_out_total": 1,
  "sink_mqtt_0_0_messages_processed_total": 1,
  "sink_mqtt_0_0_process_latency_us": 73,
  "sink_mqtt_0_0_buffer_length": 0,
  "sink_mqtt_0_0_last_invocation": "2024-06-20T12:09:44.899",
  "sink_mqtt_0_0_exceptions_total": 0,
  "sink_mqtt_0_0_last_exception": "",
  "sink_mqtt_0_0_last_exception_time": 0
}
```

再发一条测试数据如下。

```json
{
  "ts":1617180020000, 
  "frames":[
    {"id":485, "bus":5,"d":0,"t":1617179000000,"data":"000000000000AB00"},
    {"id":555,"bus":6,"d":0,"t":1617179000100, "data":"5465737400000000"}
  ]
}
```

其中 id 485 的 data 前半部分均为 0，解析后的 Angle 值应为 0，不满足条件。再次查看规则的状态指标，应该可见 "op_3_filter_0_records_out_total" 的值小于 records_in ，即有数据不满足条件被过滤了。

后续可通过 MQTT 客户端或我们的数据工具，发送更多的数据。

### 云端数据接

根据 [规则定义](http://localhost:63342/markdownPreview/699592024/markdown-preview-index-316530953.html?_ijt=73lidq25r5ri4lkqeguv4qc0e8#编写规则)，我们处理后的数据发送到**云端** MQTT broker 的 `result/angle` 中。通过 MQTTX， 我们可以订阅该主题，实时查看采集结果。在车端继续发送模拟数据，云端应当能接收到如下的采集数据：

```json
{
  "acceleration": 1,
  "angle": 84,
  "rpm": 0,
  "speed": 64.92
}
```

### 更改数据处理逻辑

- 对上述的规则做简单的更改，调用 sdv-platform 相关 API，实现数据处理逻辑的更新，展示如何通过简单的方式来修改数据处理的逻辑

在复杂的应用中，业务场景变更是经常发生的事。sdv-platform 可以通过 API 对规则进行热更新，无需 OTA 实现数据处理逻辑的更改。在前文的规则中，只要当转弯角度超过 10 我们即进行数据采集处理。然而，当用户倒车时，可能会有原地转弯角度大的情况，并不满足业务方的需求，可能会对后续云端处理，模型训练产生干扰。因此，我们可以对规则 SQL 进行迭代，当角度大于 10 且速度大于 20 时才进行采集，提高数据采集质量。

可以通过 sdv-platform 下发规则更改请求，规则如下所示：

```json
{
  "id": "ruleFetch",
  "name": "采集转弯数据",
  "sql": "SELECT DRIVE.Angle as angle, DRIVE.Speed as speed, DRIVE.Acceleration as acceleration, ENGINE.temperature as engineTemp, ENGINE.RPM as rpm FROM canDemo WHERE DRIVE.Angle > 10 AND DRIVE.Speed > 20",
  "actions": [
    {
      "mqtt": {
        "server": "tcp://remote:1883",
        "topic": "result/angle",
        "sendSingle": true
      }
    }
  ]
}
```

```shell
curl --location --request POST 'http://127.0.0.1:8082/api/edgeservices/config' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer ${auth token}' \
--data-raw '{
    "ids": [
        "${vin}"
    ],
    "method": "POST",
    "path": "/rules",
    "body": "{\n  \"id\": \"ruleFetch\",\n  \"name\": \"采集转弯数据\",\n  \"sql\": \"SELECT DRIVE.Angle as angle, DRIVE.Speed as speed, DRIVE.Acceleration as acceleration, ENGINE.temperature as engineTemp, ENGINE.RPM as rpm FROM canDemo WHERE DRIVE.Angle > 10 AND DRIVE.Speed > 20\",\n  \"actions\": [\n    {\n      \"mqtt\": {\n        \"server\": \"tcp://remote:1883\",\n        \"topic\": \"result/angle\",\n        \"sendSingle\": true\n      }\n    }\n  ]\n}"
}'
```

规则更改后立即生效，可以在云端查看采集的数据，speed 小于 20 的数据应该不再采集上传。

## 全量数据落盘

全量数据落盘功能可以在车辆运行过程中接入数据总线的数据滚动保存到本地的存储设备中，改工的流程如下所示。

![full-data-drop](_assets/full-data-drop.png)

1. 消息流入
2. 消息存入内存中
3. 当 RingBus 中数据满时执行落盘动作

Bus 中配置了exchange用于处理特定topic的消息，并将消息转入到 ringbus 内存中，当 ringbus 数据存满时会进行落盘操作，落盘生成 parquet 文件。

### 数据总线配置文件介绍

```
# #====================================================================
# # Exchange configuration for Embedded Messaging Queue
# #====================================================================
# # Initalize multiple MQ exchanger by giving them different name (mq1)
exchange_client.mq1 {
        # # Currently NanoMQ only support one MQ object. URL shall be exactly same.
        exchange_url = "tcp://127.0.0.1:10000"
        # # exchanges contains multiple MQ exchanger
        exchange {
                # # MQTT Topic for filtering messages and saving to queue
                topic = "canudp",
                # # MQ name
                name = "exchange_no1",
                # # MQ category. Only support Ringbus for now
                ringbus = {
                        # # ring buffer name
                        name = "ringbus",
                        # # max length of ring buffer (msg count)
                        cap = 1000,
                        # #  0: RB_FULL_NONE: When the ringbus is full, no action is taken and the message enqueue fail
                        # #  1: RB_FULL_DROP: When the ringbus is full, the data in the ringbus is discarded
                        # #  2: RB_FULL_RETURN: When the ringbus is full, the data in the ringbus is taken out and returned to the aio
                        # #  3: RB_FULL_FILE: When the ringbus is full, the data in the ringbus is written to the file
                        #
                        # # Value: 0-4
                        # # Default: 0
                        fullOp = 2
                }
        }
} 

# #====================================================================
# # Parquet configuration (Apply to Exchange/Messaging_Queue)
# #====================================================================
parquet {
        # # Parquet compress type.
        # #
        # # Value: uncompressed | snappy | gzip | brotli | zstd | lz4
        compress = uncompressed
        # # Encryption options
        # encryption {
        #         # # Set a key retrieval metadata.
        #         # #
        #         # # Value: String
        #         key_id = kf
        #         # # Parquet encryption key.
        #         # #
        #         # # Value: String key must be either 16, 24 or 32 bytes.
        #         key = "0123456789012345"
        #         # # Set encryption algorithm. If not called, files
        #         # # will be encrypted with AES_GCM_V1 (default).
        #         # #
        #         # # Value: AES_GCM_CTR_V1 | AES_GCM_V1
        #         type = AES_GCM_V1
        # }
        # # The dir for parquet files.
        # #
        # # Value: Folder
        dir = "./parquet"
        # # The prefix of parquet files written.
        # #
        # # Value: string
        file_name_prefix = "nanomq"
        # # Maximum rotation count of parquet files.
        # #
        # # Value: Number
        # # Default: 5
        file_count = 5
        # # The max size of parquet file written.
        # #
        # # Default: 10M
        # # Value: Number
        # # Supported Unit: KB | MB | GB
        file_size = 1KB
        # # The max number of searches per second.
        # #
        # # Default: 5
        # # Value: Number
        limit_frequency = 5
}
```

### 向特定主题发送消息（配置示例中为canudp）

- 发送 MQTT 消息到 canudp 主题。下面以 emqtt_bench 为例，持续发送数据

  ```
  $ emqtt_bench pub -p 1883 -i 1 -I 3 -c 1 -s 100 -t canudp -V 5
  ```

### 检查落盘数据是否生成

- 根据上面的配置可以观测到 Ringbus 配置的容量为1000条消息，所以每流入Bus 一千条消息会触发一次数据落盘，具体根据数据内容的大小以及 file_size 的限制，可能会生成多个文件。数据落盘文件在 parquet 目录。

```
total 20K
-rw-r--r-- 1 root root 1.2K May 30 15:39 nanomq_canudp_08d1fc56c9ebaf5eaeccdf9fbf8ed7df-1717054759843~1717054759879.parquet
-rw-r--r-- 1 root root 1.2K May 30 15:39 nanomq_canudp_52a5dfea2fd844c443c673cea04e1402-1717054759981~1717054760008.parquet
-rw-r--r-- 1 root root 1.2K May 30 15:39 nanomq_canudp_606b60e89326bcba7e30c0e93ccb2494-1717054759879~1717054759912.parquet
-rw-r--r-- 1 root root 1.2K May 30 15:39 nanomq_canudp_6c22bf89dc2e67ba3c0ac1003a6033c8-1717054759945~1717054759981.parquet
-rw-r--r-- 1 root root 1.2K May 30 15:39 nanomq_canudp_72c6d7c788d3d5df8c78ecbe29586868-1717054759912~1717054759945.parquet
```

## 总结

本文简单介绍了车云数据闭环软件的安装，展示了如何实现一个简单的端到端场景，包含了数据接入、处理逻辑的实现和修改，数据的存储，以及车云数据传输等，通过车云的控制通道可以随时对业务逻辑进行实时更新，满足了车辆预测性维护、智能驾舱和自动驾驶等场景下对灵活的数据采集和处理的功能。如读者需要了解更多的高级功能，请参考下面扩展阅读部分内容。

## 扩展阅读

- 车端数据源扩展
- 数据处理函数和功能
- sdv-platform 对接后台大数据系统
- 车端软件的资源使用监控，日志分析等
- API 接口说明文档
- AI 推理功能扩展