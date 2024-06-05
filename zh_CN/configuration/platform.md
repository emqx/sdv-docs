# SDV Platform 配置说明

sdv-platform 的配置是基于 yaml 文件，允许通过更新文件进行配置。对其进行修改需要重新启动 sdv-platform实例。

## 服务器

```yaml
server:
  port: 8082
```

- `port`：sdv-platform 服务器的端口号，默认值为 8082。

## 数据库

```yaml
db:
  host: 127.0.0.1
  user: root
  password: "root"
  dbname: sdv
  port: 3307
  timeout: 10s
  autoMigrate: true
  timezone: "Local"
  logLevel: info
  slowThreshold: "200ms"
```

- `host`：数据库的主机地址
- `user`: 数据库的用户名
- `password`: 数据库的密码
- `dbname`: 数据库的名称
- `port`: 数据库的端口号
- `timeout`: 数据库连接超时时间
- `autoMigrate`: 应用启动时是否自动迁移同步数据库结构
- `timezone`: 数据库的时区
- `logLevel`: 数据库日志级别
- `slowThreshold`: 数据库慢查询阈值

## 日志

```yaml
logger:
  mode: console
  format: text
  path: /tmp/sdv-platform.log
  level: info
  maxSize: 10
  maxAge: 7
  maxBackups: 3
```

- `mode`: 指定日志模式。默认设置为文件模式，支持 `file|console` 两种。
- `format`: 指定日志格式。默认设置为文本格式，支持 `text|json` 两种。
- `path`: 指定日志文件的路径。默认为为 `/tmp/sdv-platform.log`。可以根据需要修改日志文件的路径。
- `level`: 指定日志级别，默认设置为 info 级别 支持 `debug|info|notice|warn|error|fatal` 级别。
- `maxSize`: 指定日志文件的最大文件大小（MB），超过此大小后会进行日志文件轮转。
- `maxAge`: 指定日志文件的最大保存时间（天），超过此时间后会进行日志文件删除。
- `maxBackups`: 指定保留的旧日志文件的最大数量。超过数量将进行覆盖。

## mqtt

```yaml
mqtt:
  address: tcp://127.0.0.1:1883
  username: "admin"
  password: "public"
  maxReconnectInterval: 3
  connectTimeout: 8
  cleanSession: true
  url: "http://127.0.0.1:18083"
  loginAccount: "admin"
  loginPassword: "Public123"
```

- `address`: mqtt 服务器的地址
- `username`: mqtt 服务器的用户名
- `password`: mqtt 服务器的密码
- `maxReconnectInterval`: mqtt 重连的最大间隔时间
- `connectTimeout`: mqtt 连接超时时间
- `cleanSession`: mqtt 是否清除会话
- `url`: emqx 服务器的 url
- `loginAccount`: emqx 登录账号
- `loginPassword`: emqx 登录密码

## 指标

```yaml
metric:
  enable: true
  pushGatewayUrl: http://127.0.0.1:31901
```

- `enable`: 是否开启指标监控，收集上报到云端的指标
- `pushGatewayUrl`: 指标推送地址（地址 emqx 需可访问通过）