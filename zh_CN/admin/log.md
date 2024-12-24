# 管理日志文件

车端sdv-flow 的日志来源包括三个模块，分别是数据总线，流处理引擎，边缘协调同代理，根据配置文件的内容，日志文件有不同的处理方式。

```yaml
log:
  mode: console
  level: info
  file: log/sdv-flow.log
  maxSize: 10  # maximum size in megabytes of the log file before it gets rotated
  maxBackups: 5 # MaxBackups is the maximum number of old log files to retain
  enableSyslog: true
  syslogType: "unixgram" # udp4 or unixgram
  syslogAddr: /tmp/sdv-flow.socket
  
syslogServer:
  enable: true
  level: info
  addr: "/tmp/sdv-flow.socket" # "localhost:10514" or socket file
  type: "unixgram"
  file: "/tmp/sdv-flow/log/sdv-flow.log"
  maxSize: 102400  # maximum size in bytes of the log file before it gets rotated
  maxBackups: 5 # MaxBackups is the maximum number of old log files to retain
  maxAge: 10
  enableLogcat: false
  enableLogUpload: true
```

- 当enableSyslog 为false 时，三个模块的日志文件是独立的，分别生成在各个模块的log 文件夹下，日志级别大小等限制同样在各自的配置文件中定义。
- 当enableSyslog 为 true 时，三个模块的日志将以syslog 的方式一同写入syslogServer.file 定义的文件夹下，日志级别大小等限制则在syslogServer 下定义，同时当日志文件大小满足maxSize，则会触发日志文件周期上传。

## 日志配置

目前云端支持通过api 下发配置修改车端的日志级别和大小，示例如下：

**POST /api/edgeservices/config**

```
{
 "ids": [
 "ecs-2c63"
 ],
 "method": "POST",
 "path": "/configs/log",
 "category": 2,
 "body": "{\"level\": \"debug\",\"module\": \"all\",\"maxSizeBytes\":\"102400\"}"
}
```

- module：重启的模块，支持 streamEngine,messageBus,agent ,all ,当enableSyslog 为 true 时，moudle 仅支持all
- level: 日志级别,支持 debug,info ,warn ,error
- 下发成功后，车端会立即自动重启，以使配置生效

## 日志周期上传

sdv-flow 在运行过程中会持续生成日志内容。根据配置，当单个日志内容的大小达到设定的`syslogServer.maxSize`阈值时，会触发日志内容的落盘操作。落盘后的日志文件将被存储在`syslogServer.file`配置项所定义的文件夹下。这一机制确保了日志数据的持久化存储，方便后续的查询与分析。

为了提高数据传输效率和节省云端存储空间，落盘后的日志文件会被自动压缩。压缩完成后，系统将启动日志文件的上传流程，将压缩后的日志文件内容上传至云端。上传过程中，系统会确保数据的完整性和安全性，避免在传输过程中出现数据丢失或损坏的情况。

一旦日志文件成功上传至云端，系统会自动删除车端存储的压缩后的日志文件。这一操作有助于释放车端的存储空间，确保系统能够持续稳定地运行，同时避免因存储空间不足导致的日志数据丢失问题。

日志文件的上传遵循特定的主题命名规则，上传主题为`$file/upload/logfile/periodic/{vin}/{md5sum}/{filename}`。其中：

- `{vin}`：代表车辆识别号，用于标识日志文件所属的车辆。
- `{md5sum}`：是日志文件内容的MD5校验和，用于验证上传文件的完整性和一致性。
- `{filename}`：是日志文件的名称，包含了日志文件的相关信息，如生成时间、日志级别等。

这种命名方式有助于在云端对日志文件进行有效的分类和管理，方便后续的日志查询、分析和问题排查工作。

通过SDV-Flow的日志文件周期上传机制，客户可以轻松地将车端生成的日志数据传输至云端，实现日志数据的集中管理和分析，为车辆的运行监控和问题排查提供有力支持。

## 日志选择上传

sdv-flow 软件在运行过程中会持续生成日志文件，记录系统运行状态、资源占用情况、异常事件等关键信息。这些日志文件对于排查问题、了解系统运行情况具有重要意义。随着时间的推移，日志文件数量会逐渐增多，因此需要一种便捷的方式来获取和上传指定的日志文件。日志文件选择上传流程：

1. 云端提供了专门的API接口用于获取日志文件路径及对应的时间范围。

   **GET /api/edgeservice/proxy/agent/{vin}/loglist** 

   ```
   {
       "paths": [
           {
               "name": "syslog",
               "path": "/tmp/sdv-flow/",
               "files": [
                   {
                       "name": "sdv-flow.log",
                       "start": "2024-11-05 17:57:58",
                       "end": "2024-11-06 11:34:47"
                   },
                   {
                       "name": "sdv-flow.log.20241105164132~20241105164232",
                       "start": "2024-11-05 16:41:32",
                       "end": "2024-11-05 16:42:32"
                   },
                   {
                       "name": "sdv-flow.log.20241105164232~20241105164532",
                       "start": "2024-11-05 16:42:32",
                       "end": "2024-11-05 16:45:32"
                   }
               ]
           }
   }
   ```

2. 当需要对系统进行问题排查或了解特定时间段内的运行情况时，首先应明确排查的具体需求，比如是针对某个功能模块的异常、资源占用峰值等。根据排查需求，确定需要获取日志文件的时间范围。根据时间范围确定需要上传哪些文件。

3. 通过云端API下发上传命令，指定之前选择的日志文件。API会将上传命令发送至车端。

   **POST /api/edgeagents/{vin}/fileupload**

   ```
   {
       "request_id": "22222222",
       "files": [
           "/tmp/sdv-flow/sdv-flow.log",
           "/tmp/sdv-flow/sdv-flow.log.20241105164232~20241105164532"
       ],
       "filenames": [
           "sdv-flow.log",
           "sdv-flow.log.20241105164232~20241105164532"
       ],
       "delete": [
           -1,
           -1
       ]
   }
   ```

   - request_id :请求id，任意字符串,不能含有/，用于api查询下发结果
   - files ：待上传的文件列表，绝对路径，可以参考loglist api 的返回path +文件名
   - filenames：文件名
   - delete ：删除设置数组，//默认值：-1，长度需要和文件files 相同
     - ＜0:传输文件成功后不删除文件
     - =0：传输文件成功后立即删除文件
     - 0：传输文件成功后n秒（最多7天）后删除文件

4. 日志文件的上传遵循特定的主题命名规则，上传主题为`$file/upload/logfile/manual/{requestId}/{vin}/{md5sum}/{filename}`。其中：

- `{requestId}`：代表下发时指定的请求id
- `{vin}`：代表车辆识别号，用于标识日志文件所属的车辆。
- `{md5sum}`：是日志文件内容的MD5校验和，用于验证上传文件的完整性和一致性。
- `{filename}`：是日志文件的名称，包含了日志文件的相关信息，如生成时间、日志级别等。