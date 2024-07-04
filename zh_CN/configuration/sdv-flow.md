sdv-flow 的配置是基于 yaml 文件，允许通过更新文件进行配置。对其进行修改需要重新启动 sdv-flow 实例。

# 配置文件

本文档提供了关于配置文件的详细说明和使用教程，帮助你了解和配置系统的不同模块，服务以及日志设置。

## 日志配置

配置文件中的日志部分包含了系统日志的相关设置。
```yaml
log:
  mode: file
  level: info
  file: log/sdv-flow.log
  maxSize: 20  # maximum size in megabytes of the log file before it gets rotated
  maxBackups: 5 # MaxBackups is the maximum number of old log files to retain
  enableSyslog: false
  syslogType: "unixgram" # udp4 或 unixgram
  syslogAddr: /dev/log
```

- `mode`: 指定日志模式。默认设置为文件模式，支持 `file|console` 两种。
- `level`: 指定日志级别，默认设置为 info 级别 支持 `debug|info|notice|warn|error|fatal` 级别。
- `file`: 指定日志文件的路径。默认为为 `log/sdv-flow.log`。可以根据需要修改日志文件的路径。
- `maxSize`: 指定日志文件的最大文件大小（MB），超过此大小后会进行日志文件轮转。
- `maxBackups`: 指定保留的旧日志文件的最大数量。超过数量将进行覆盖。
- `enableSyslog`: 是否启用 Syslog 日志记录，布尔值。
- `syslogType`: Syslog 的类型，可以是 udp4 或 unixgram。
- `syslogAddr`: Syslog 服务器地址，当 enableSyslog 为 true 时生效。

## Syslog Server 配置
配置文件中的`syslogServer`部分包含了系统中不同模块的配置信息。
```yaml
syslogServer:
  enable: false
  addr: "/tmp/sdv-flow.socket" # "localhost:10514" or socket file
  type: "unixgram"
  file: "/tmp/sdv-flow/sdv-flow.log"
  maxSize: 20  # maximum size in megabytes of the log file before it gets rotated
  maxBackups: 5 # MaxBackups is the maximum number of old log files to retain
  maxAge: 10
```
- `enable`: 是否启用 Syslog 服务器，布尔值。
- `addr`: Syslog 服务器地址，可以是本地主机和端口如 localhost:10514，也可以是 socket 文件路径。
- `type`: Syslog 服务器的类型，可以是 udp4 或 unixgram。
- `file`: Syslog 日志文件路径。
- `maxSize`: 日志文件的最大大小（以兆字节为单位）。当日志文件达到此大小时，将进行日志轮换。
- `maxBackups`: 保留的旧日志文件的最大数量。
- `maxAge`: 日志文件保留的最大天数。



## 模块配置

配置文件中的`modules`部分包含了系统中不同模块的配置信息。

```yaml
moduleOnFailure: exitModule
modules:
  # The 'modules' section contains the configuration for various modules in the system.
  - name: example # Specifies the name of the module.
    enabled: false  # Determines whether the module is enabled (true) or disabled (false).
    mechanism: periodic  # Defines the execution mechanism of the module. Options: once, lifecycle, periodic.
    type: shell  # Specifies the type of the module. Options: built-in, shell.
    interval: 10  # Specifies the interval (in seconds) for periodic modules.
    maxRunTime: 10  # Specifies the maximum runtime for the module in seconds. Mechanism lifecycle isn's needed.
    cmd: "echo 'hello, world'"  # Specifies the command to execute for shell-type modules.
    version: 1  # Indicates the version of the module.
    dependencies: []  # Lists the dependencies of the module.
    run-on: init  # Determines when the module should run. Options: early-init, init, finit.


  # Built-in modules only have three options: name, enabled, and type.
  - name: getvin
    enabled: true
    type: built-in
```

`moduleOnFailure`: 当模块失败时的行为。可以设置为 exitModule、terminateAll 或 restartModule。
- `exitModule`: 退出当前模块。
- `terminateAll`: 终止所有模块。
- `restartModule`: 重启当前模块。

**注意: 此处也包括对 NanoMQ 和 Ekuiper 的控制。**

每个模块的配置包括以下选项：

- `name`: 模块的名称。指定模块的名称。
- `enabled`: 模块的启用状态。设置为`true`表示启用该模块，设置为`false`表示禁用该模块。
- `mechanism`: 模块的执行机制。可选值有`once`、`lifecycle`和`periodic`。`once`表示只执行一次，`lifecycle`表示根据系统的生命周期执行，`periodic`表示定期执行。
- `type`: 模块的类型。可选值有`built-in`和`shell`。`built-in`表示内置模块，`shell`表示Shell脚本模块。
- `interval`：对于周期性模块，指定执行的间隔时间（以秒为单位）。
- `maxRunTime`：指定模块的最长运行时间（以秒为单位）。生命周期执行机制下的模块不需要此属性。
- `cmd`: Shell脚本模块的执行命令。只有当`type`设置为`shell`时才有效。
- `version`: 模块的版本号。
- `dependencies`: 模块的依赖关系列表。列出模块所依赖的其他模块。
- `run-on`: 模块的运行时机。可选值有`early-init`、`init`和`finit`。`early-init`表示早期初始化时运行，`init`表示初始化时运行，`finit`表示最终初始化时运行。

## 服务配置

配置文件中的`server`部分包含了系统中其他功能点的配置。

```yaml
server:
  port: 14260
  parquet:
    path: /tmp/parquet
    days: 30
  heartbeat:
    enable: true
    interval: 15
    metricsEnable: true
#    common or debug
    level: common
```

- `port`：Api 默认端口
- `parquet`：每次 sdv-flow 启动时将按配置项清除 parquet 文件。
  - `path`：待清除的 parquet 文件夹的路径。
  - `days`：parquet 文件保留的天数。
- `heartbeat`：sdv-flow 将按配置向云端上报心跳信息
  - `enable`：开启还是关闭上报。
  - `interval`：上报间隔。
  - `metricsEnable`：上报心跳时，是否上报指标信息。
  - `level`：`metricsEnable`为 true 时，指标的内容级别，common 为基础信息，debug 为全部信息，方便调试。

