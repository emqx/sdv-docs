# CAN 数据源

<span style="background:green;color:white;padding:1px;margin:2px">stream source</span>
<span style="background:green;color:white;padding:1px;margin:2px">scan table source</span>

CAN 数据源通过 socketCan 连接到 CAN 总线并读取 CAN 帧。

## 数据流通用属性

创建数据流时可使用一系列通用的属性，列表请参考 [流通用属性](https://ekuiper.org/docs/zh/latest/guide/streams/overview.html#%E6%B5%81%E5%B1%9E%E6%80%A7)。
CAN 数据源支持其中一部分属性，如下所示。不支持的属性配置后会忽略。

- FORMAT: 格式，必须为 `can`.
- SCHEMAID: 指定 DBC 文件的路径。
- CONF_KEY: 指向配置文件中定义的配置项
- SHARED: 是否被规则共享。CAN 格式解析负载较重，因此通常会设置流为共享，以避免重复解析。

这些属性在流创建SQL中使用，请查看[示例](#示例)了解详细信息。

## 配置

数据源可以通过[环境变量](https://ekuiper.org/docs/zh/latest/configuration/configuration.html#environment-variable-syntax)、[REST API](https://ekuiper.org/docs/zh/latest/api/restapi/configKey.html) 或配置文件进行配置，本节将介绍配置文件的使用方法。

Can 数据源的配置文件位于 `/etc/sources/can.yaml`。

**示例**

```yaml
default:
  # The network type of the CAN bus, can be can or udp
  network: can
  # The address of the CAN bus
  address: can0
```

- network: 定义 CAN 总线网络类型，可为 can 或者 udp 。
- address: CAN 总线的接口地址。 可通过命令 `ip link show` 查询系统中的网络接口，选择其中的 CAN 虚拟或物理接口地址。

## 示例

**注意**: CAN 数据源必须使用 `can` 格式。

```sql
create stream canDemo () WITH (TYPE="can", CONF_KEY="default", FORMAT="can", SHARED="true", SCHEMAID="dbc")
```

详细教程请参阅 [processing data from CAN BUS](./overview.md#scenario-1-connect-to-can-bus)。
