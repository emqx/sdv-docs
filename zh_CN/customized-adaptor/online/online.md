# 上线上报状态

```json
{
    "rules": {
        "rule1": "running",
        "rule2": "stopped"
    },
    "customProperties": {
        "signalDefinition": {
            "version": "v1.1.0"
        },
        "cs": "E01",
        "nc": "carconfig746",
        "dataSpan": {
            "canspi": {
                "start": "1723017850443",
                "end": "1723018793178"
            },
            "signal": {
                "start": "1723017850443",
                "end": "1723018793178"
            },
            "battcell": {
                "start": "1723017850443",
                "end": "1723018793178"
            },
            "batttsnsr": {
                "start": "1723017850443",
                "end": "1723018793178"
            }
        }
    },
    "clientid": "localhost",
    "log": {
        "level": {
            "agent": "error",
            "streamEngine": "error",
            "messageBus": "error"
        }
    },
    "threshold": {
        "enable": true,
        "cpu": "80%",
        "memory": "10",
        "action": "restart",
        "dutation": 10
    },
    "ts": 1725520583
}
```

### 1. `rules` 字段

该字段包含一组规则及其状态，用于描述车辆当前遵循的规则及其执行情况。

- **rule1**：表示规则1，当前状态为`"running"`，意味着规则1正在正常运行。
- **rule2**：表示规则2，当前状态为`"stopped"`，意味着规则2已停止运行。

### 2. `customProperties` 字段

该字段包含自定义属性，用于提供额外的车辆相关信息。

- `signalDefinition`

  ：信号定义信息。

  - **version**：信号定义的版本号，当前为`"v1.1.0"`。

- **cs**：车辆所属地域，当前值为`"E01"`。

- **nc**：车辆所属国家，当前为`"carconfig746"`。

- `dataSpan`

  ：数据时间跨度信息，包含多个子字段。

  - **canspi**：CAN SPI数据的时间范围，起始时间为`"1723017850443"`，结束时间为`"1723018793178"`。
  - **signal**：信号数据的时间范围，起始时间和结束时间与`canspi`相同。
  - **battcell**：电池单元数据的时间范围，起始时间和结束时间与`canspi`相同。
  - **batttsnsr**：电池温度传感器数据的时间范围，起始时间和结束时间与`canspi`相同。

### 3. `clientid` 字段

表示客户端标识，值为 `vin` 码。

### 4. `log` 字段

包含日志级别信息，用于描述不同模块的日志记录级别。

- `level`

  ：日志级别设置。

  - **agent**：代理模块的日志级别为`"error"`，表示仅记录错误级别的日志。
  - **streamEngine**：流引擎模块的日志级别也为`"error"`。
  - **messageBus**：消息总线模块的日志级别同样为`"error"`。

### 5. `threshold` 字段

包含阈值监控配置信息，用于描述资源占用的阈值及其相关设置。

- **enable**：是否启用阈值监控，当前值为`true`，表示已启用。
- **cpu**：CPU占用阈值，设置为`"80%"`，即当CPU占用超过80%时触发相关操作。
- **memory**：内存占用阈值，设置为`"10"`，单位为MB，即当内存占用超过10MB时触发相关操作。
- **action**：超过阈值时执行的操作，当前设置为`"restart"`，表示将重启相关服务或进程。
- **dutation**：持续时间，设置为`10`，单位为秒，表示当CPU或内存占用超过阈值的情况持续10秒后，才会执行`action`中设定的操作。

### 6. `ts` 字段

表示时间戳，当前值为`1725520583`，用于记录数据上报的时间。