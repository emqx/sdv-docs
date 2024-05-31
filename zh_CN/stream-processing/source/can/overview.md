# 分析 CAN 总线数据

数据流计算引擎可以连接到 CAN 总线，并把原始数据解析为结构化数据。通常，它提供以下这些功能：

- [can 数据源](./can_source.md): 通过 socketCAN 读入 CAN 总线数据。
- can 相关序列化格式：
  - [can 格式](./can_format.md): 解析CAN帧为物理键值对，以便由流计算引擎处理。用户可以指定DBC文件来解析CAN帧。
  - [canjson 格式](./canjson_format.md): 将 JSON 字符串中的批量 CAN 帧解析为流计算引擎要处理的物理键值对。用户可以指定DBC文件来解析CAN帧。

其中，source 数据源是流的类型，负责连接并从外部系统读取数据，如 MQTT、视频和 socketCAN。而 format 格式负责序列化/反序列化，如 JSON 格式、protobuf 格式等。这两个概念是相关联且解耦的。用户可以自由组合 source 和format 以匹配实际场景。例如，MQTT source 与 can format 结合可以处理通过 MQTT 协议传输 CAN 帧的情况。

## 快速开始

我们将快速介绍两个场景。首先，我们将创建一个规则，直接连接到 CAN 总线，并读取和处理 CAN 总线数据。其次，我们将处理通过 MQTT 传递的 CAN 总线数据。由于安全或其他原因，我们可能不被允许直接连接到 CAN 总线，多亏了 SDV flow 的灵活设计，我们仍然用相同的 SQL 语句可以处理 CAN 总线数据。

### 前提条件

DBC 文件定义了 CAN 总线上的信号。我们使用 DBC 文件将 CAN 总线数据解码成可读的信号。在运行示例之前，您需要准备 DBC 文件并将它们放置在 `dbc` 文件夹中。请查看文档 [DBC](./dbc.md) 以获取更多信息和配置提示。

### 场景1：直连 CAN 总线

SocketCAN是Linux内核中的一种网络协议实现，它为与控制器局域网络（CAN）设备通信提供了基于套接字的接口。它是Linux内核的一部分。要运行此场景，我们需要准备一台测试机器，并满足以下要求：

- Linux操作系统（Ubuntu或Debian等）
- CAN接口，可以是虚拟接口或物理接口。
- 
如果您不确定是否满足这些要求，请按照下一节的说明进行设置。

#### 设置 CAN 接口

如果您使用真实硬件连接到 CAN 总线，您将拥有原生的 CAN 接口。使用`ip link show`命令列出所有接口，类型为`link/can`的接口就是 CAN 接口。

如果您有一个物理 CAN 接口，您应该能在列表中看到该接口。记住它的名称，这将在下一步中使用。

如果您没有真实的 CAN 接口，您可以使用虚拟 CAN 接口。以 Ubuntu 为例，我们可以通过以下命令启用虚拟 CAN 接口并创建一个名为 `can0` 的 vcan 接口：

```bash
sudo modprobe vcan
sudo ip link add dev can0 type vcan
sudo ip link set up can0
```

通过命令 `ip link show` 查询系统中所有的接口。应该可以看到名为 `can0` 的接口。

#### 收发 CAN 数据

安装 `can-utils` 用于手动收发 CAN 数据。

```bash
sudo apt install can-utils
```

然后，我们可以通过以下命令接收并打印原始的CAN数据：

```bash
candump can0
```

在另一个终端，我们可以发送CAN数据进行测试：

```bash
cansend can0 123#1122334455667788
```

其中，`123` 是CAN ID，`1122334455667788` 是数据负载。请确保数据能够在第一个终端打印出来。到此为止，我们的 CAN 接口已经准备好了。

在下一节中，我们将使用 cansend 向流处理引擎发送测试数据。

#### 创建 CAN 数据流

我们提供了 [REST API](https://ekuiper.org/docs/zh/latest/api/restapi/overview.html) 用于管理流处理规则。要使用 REST API，我们建议使用 Postman 或其他任意 HTTP 客户端工具发送 HTTP 请求。在本文档中，我们将为每个步骤指定 HTTP 方法、URL 和请求体。

首先，我们需要创建一个流以连接到虚拟 can 接口 can0。流的定义如下：

```sql
create stream canDemo () WITH (TYPE="can", CONF_KEY="default", FORMAT="can", SHARED="true", SCHEMAID="dbc")
```

- `TYPE="can"`: 流的类型为 `can`, 用于通过 socketCan 连接到 CAN 总线。
- `CONF_KEY="default"`: 配置键为 `default`, 它将指向配置文件中的默认配置。配置文件位于 `etc/sources/can.yaml` 其中定义了 CAN 总线地址 `can0`。若需要连接另外的地址，请修改该配置。
- `FORMAT="can"`: 数据格式为 `can`, 该格式可解析二进制的 CAN 帧。
- `SHARED="true"`: 数据流可为多个规则共享，这样多个规则可以共用解析结果，无需多次重复解析。
- `SCHEMAID="dbc"`: 配置流的 schema，即 DBC 文件的路径。

使用 REST API 创建流，请求描述如下：

```http request
POST http://{{ekuiper_host}}/streams
Content-Type: application/json

{
  "sql": "create stream canDemo () WITH (TYPE=\"can\", CONF_KEY=\"default\", FORMAT=\"CAN\", SHARED=\"TRUE\", SCHEMAID=\"dbc\")"
}
```

该格式表示：

- `POST` : HTTP 请求为 `POST` 类型。
- `http://{{ekuiper_host}}/streams`: 请求的 URL。使用中，将 `{{ekuiper_host}}` 替换为您的流处理引擎部署地址。
- `Content-Type: application/json`: 添加一个 HTTP header 来指明请求体是一个 JSON 字符串。
- `{"sql":...}`: 请求体 JSON 字符串。

命令执行成功后，应该已创建流 canDemo 。您可以通过以下请求获取流列表来验证。确保 canDemo 在列表中。

```http request
GET http://{{ekuiper_host}}/streams
Content-Type: application/json
```

#### 创建第一个规则

接下来，我们可以创建一个规则来收集所有数据。我们创建一个名为 `canAll` 的最简单的规则，从流 `canDemo` 中选择所有数据，并将其发送到公共 EMQX 代理 `tcp://broker.emqx.io:1883` 的 MQTT 主题` result/can/all` 中。您可以通过更改规则 MQTT 动作中的服务器属性来使用您自己的 EMQX 代理。

```http request
POST http://127.0.0.1:9081/rules
Content-Type: application/json

{
  "id": "canAll",
  "sql": "Select * From canDemo",
  "actions": [
    {
      "mqtt": {
        "server": "tcp://broker.emqx.io:1883",
        "topic": "result/can/all",
        "sendSingle": true
      }
    }
  ]
}
```

#### 测试规则


使用 MQTT 客户端 [MQTTX](https://mqttx.app/)，连接到 `tcp://broker.emqx.io:1883` 或规则动作中指定的代理地址，订阅 `result/can/all` 主题，并准备查看规则的输出结果。

转到宿主机终端，并通过 cansend 命令向 can 接口 can0 发送测试数据：

```bash
cansend can0 586#5465737400000000
```

确保数据能够匹配您的 DBC 文件。ID 必须在 DBC 文件中定义，并且数据负载的长度必须与 DBC 文件中定义的相同。

**请注意，如果 ID 不在 DBC 文件中，数据将被忽略。**

检查 MQTTX 中的输出，您应该收到类似的消息：

```json
{
  "MessageName1": {
    "VBBrkCntlAccelPedal": 0,
    "VBTOSLatPstn": 87.125,
    "VBTOSLonPstn": 168.75
  }, 
  "MessageName2": {
    "VBTOSObjID": 0,
    "VBTOSTTC": 46.4
  }  
}
```

请注意，解析的字段有两层。第一层是消息名称，第二层是信号名称。这些名称在 DBC 中定义。在规则中，您可以直接使用可读的信号。例如，选择特定的信号：`SELECT messageName1.signal1, messageName2.signal34 FROM canDemo`, 享受探索解析信号的强大处理能力的乐趣。

### 场景2：处理转发的 CAN 信号

由于安全或隐私原因，我们可能不被允许直接连接到 CAN 总线。通常情况下，用户会有一个网关来接收 CAN 数据，并通过其他协议如 TCP、UDP 或 MQTT 将其发送到应用程序。网关通常会将多个 CAN 帧打包成一条消息。

在本节中，我们将以 MQTT 为例，展示如何处理来自其他协议的 CAN 数据。序列化格式可能是私有的。我们将使用 CANJSON 格式，它将多个 CAN 帧打包成一个 JSON 来发送 CAN 数据。

#### 创建处理规则

参考上一节关于如何通过 REST API 创建流和规则的内容。在这一部分，我们将只强调流/规则的内容。

首先，我们需要创建一个流来连接到 MQTT 以接收数据。流的定义如下：

```sql
create
stream mqttCanDemo () WITH (TYPE="mqtt", CONF_KEY="default", FORMAT="canjson", SHARED="true", SCHEMAID="dbc", DATASOURCE="canDemo")
```

- `TYPE="mqtt"`: 流的类型为 `mqtt`, 表示该数据流将连接 MQTT broker，订阅主题来接收数据。
- `DATASOURCE="canDemo"`: MQTT 订阅主题 `canDemo`。您可替换为另外的主题。
- `CONF_KEY="default"`: 设置配置键为默认，所有的 MQTT 键都定义于 `etc/mqtt_source.yaml`。
- `FORMAT="canjson"`: 数据格式为 `canjson`，用于将自定义的 canjson 格式数据解析为可读的信号数据。
- `SHARED="true"`: 数据流可为多个规则共享，这样多个规则可以共用解析结果，无需多次重复解析。
- `SCHEMAID="dbc"`: 配置流的 schema，即 DBC 文件的路径。

接下来，我们可以创建以下规则用于读取并打印所有解析数据到日志中。

```json
{
  "id": "canAll2",
  "sql": "Select * From mqttCanDemo",
  "actions": [
    {
      "log": {}
    }
  ]
}
```

我们创建了一个名为 `canAll2` 的最简单的规则，从流 `mqttCanDemo` 中选择所有数据并将其打印出来。它将显示原始数据被解析成可读的信号。

#### 测试规则

向 MQTT 主题 `canDemo` 发送测试数据。这些帧可以包含任意数量的 CAN 帧。确保每个 CAN 帧的 ID 都在 DBC 文件中定义。

```json
{
  "frames": [
    {
      "id": 1006,
      "data": "54657374000000005465737400000000"
    },
    {
      "id": 1414,
      "data": "5465737400000000"
    }
  ]
}
```

我们应该得到如下结构的数据：

```json
{
  "MessageName1": {
    "VBBrkCntlAccelPedal": 0,
    "VBTOSLatPstn": 87.125,
    "VBTOSLonPstn": 168.75
  },
  "MessageName2": {
    "VBTOSObjID": 0,
    "VBTOSTTC": 46.4
  }
}
```

尽管协议和格式不同，我们神奇地得到了与场景1相同的解析结构。这保证了我们可以不管数据来源如何，都可以使用相同的 SQL 语句来处理数据。只要数据内容相同，我们将总是得到相同的处理结果。

### 下一步处理

现在我们得到了解析后可读的 CAN 数据，我们可以利用数据处理能力对数据进行进一步处理，就像我们从 MQTT 或其他协议接收到的 JSON 数据一样。请查看 [eKuiper文档](https://ekuiper.org/docs/en/latest/) 了解更多您可以执行的场景。

## 进一步阅读

- [Bridging Demanded Signals From CAN Bus to MQTT by eKuiper](https://www.emqx.com/en/blog/bridging-demanded-signals-from-can-bus-to-mqtt-by-ekuiper)
- [Data Stream Processing for Software-Defined Vehicle](https://www.emqx.com/en/blog/data-stream-processing-for-software-defined-vehicle)
- [What Is Can Bus and How It Works?](https://www.emqx.com/en/blog/can-bus-how-it-works-pros-and-cons)

## 参考文档

- [CAN 数据源](./can_source.md)
- [CAN 格式](./can_format.md)
- [CANJSON 格式](./canjson_format.md)
- [DBC](./dbc.md)
