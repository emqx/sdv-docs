# 快速开始
sdv-flow 是一款车载端产品,集成了数据采集、分析处理及规则执行等功能,为数据驱动型的汽车研发和运营等业务赋能。它可以帮助企业更好地收集、分析和利用车载数据,支持数据驱动的业务决策和运营管理。
sdv-flow 作为一款车载管理软件，集成了 NanoMQ 和 kuiper 的商业版，从而实现边缘数据分析预处理，以及到云端的数据闭环功能。
本章节将带您体验从下载安装开始，到启动 sdv-flow 服务，到通过 NanoMQ 对数据进行落盘，kuiper 下发规则，以及对落盘文件的处理与上传的完整流程。

## 云端部署 emqx
- 首先在云端启动 emqx-enterprise
```bash
docker run -d --name emqx-enterprise -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx-enterprise:5.6.0
```
- 查看 emqx-enterprise 是否正常启动
```bash
docker ps -a | grep emqx-enterprise
```
## 云端部署 sdv-platform
- 下载 [docker-compose tar 包](https://github.com/emqx/sdv-platform/releases/download/0.9.0-beta.3/emqx-sdv-platform-docker-compose-installer-0.9.0-beta.3.tar.gz)并解压。
```bash
tar zxvf emqx-sdv-platform-docker-compose-installer-0.9.0-beta.3.tar.gz
```
- 解压得到 docker-compose 目录，包含如下文件
```bash
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
- 执行 `./emqx_sdv_platform_ctl --help` 获取帮助信息。
- 执行 `./emqx_sdv_platform_ctl configure` 进行配置，只需要更改 mqtt external address 选项，改为云端 broker 的 host 地址。这里假设 emqx 和 platform 在同一台机器。
```bash
$ ./emqx_sdv_platform_ctl configure
Generating docker-compose .env file
Please input EMQX SDV PLATFORM image tag (default: latest):
Please input EMQX SDV PLATFORM docker registry URL (default: docker.io):
Please input MQTT external address(which will be used for emqx agent) (default: host.docker.internal): 192.168.21.19
Please input EMQX SDV PLATFORM data volume path (default: ./datavolumes/):
stat: 无法读取 '%a' 的文件系统信息: 没有那个文件或目录
copy prometheus.yaml files ...
Generating docker-compose env file ...
Generating SDV PLATFORM config files ...
Generating ecp.yaml ...
All configurations are done.
```
![](_assets/platform-configure.png)
- 执行 `./emqx_sdv_platform_ctl start` 启动 compose 容器，结果如下：
```bash
$ ./emqx_sdv_platform_ctl start
[+] Running 4/5
 ⠋ Network docker-compose_default  Created
 ✔ Container sdv-prometheus        Started
 ✔ Container sdv-pushgateway       Started
 ✔ Container sdv-db                Started
 ✔ Container sdv-platform          Started
```
- 执行 `./emqx_sdv_platform_ctl create-user` 创建用户名密码，结果如下：
```bash
$ ./emqx_sdv_platform_ctl create-user
Please input email: sdv@emqx.io
Please input password:
Please input password again:
Creating admin sdv@emqx.io ...
Getting access token ...
{"tokenType":"Bearer","accessToken":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJFTVFYIEVDUCIsInN1YiI6IjY2ZmRiODc1IiwiZXhwIjoxNzE2OTg0NDQwLCJuYmYiOjE3MTY5NzAwNDAsImlhdCI6MTcxNjk3MDA0MCwianRpIjoiMWI5ZjhiMWYtOWJkZi00YmY4LWJhZGQtOWZhNDk3NGRhZDE0IiwidHlwIjoiQmVhcmVyIn0.jYI-FKZymyHD39gwGROWRk36DjpAtxA-c1bcx2J592JN5nnk4hxYNm3t7xeABKFruzhJK2YVDHWZa73vkRTfmMyO0ICrhaWDjsfuDxz0vKKb0TbK1ys03uneJotJqODC5TJA1rwkxcFqe7aOYTaa-ueB052nKIYZNHZiKgjTEc6TQFT-qaLLqMQeTB_mv-z1Dllh1_HWieA4FZ-Zu7aszNjh0bOd9t4V_2UTWDJJChlBcm11YTovKBbePHjLHf741OlSBErr9SvjikyikA6Py-VibM5oHzyVaQP5ekZ5-ll8cVsAQXXTNpv9OJw1GvJpXMyPuoD3Y5qZPPk-VBmT1A","expiresIn":14400,"refreshToken":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJFTVFYIEVDUCIsInN1YiI6IjY2ZmRiODc1IiwiZXhwIjoxNzE3MDU2NDQwLCJuYmYiOjE3MTY5NzAwNDAsImlhdCI6MTcxNjk3MDA0MCwianRpIjoiYTZlNGRjZmQtOGI5Yi00NDI5LThmNjMtNWM1YzlkNjA2N2MxIiwidHlwIjoiUmVmcmVzaCJ9.DGrHb7aIUVOBAakC_vrJ24FV98fhfr4kjYH5U2SOIW0tfO3vhnn_erh6euDc4TbpSlviVEHGCo66KJawjfARhwRBphBXtlEZpnnT3LuUbSnttYoYylg_rC4vMNowt2my_30KCQdsVhoOsXbyoubxWCNR2zt39DKGR1WD6jBh5OMt81DyYUWtiSaH6szDfQRVv04YJz6C9-sf0Xh9QINyRBQXHcusTN2TCvEnzBBlej2v964_2ZCq36ZFbBf80Jk1Lor-NP1iwdCZnjyG54JWbxxYtZGtK59RrPrs5O-337JliWcE5PBN3lf7hg5BWMGYfcFM1BY97yjOJemU7xw4kA","refreshExpiresIn":86400,"admin":true}
{"userId":"66fdb875","name":"admin","email":"sdv@emqx.io","mobile":"","admin":true}
Creating org default-org ...
{"id":"166322a3","name":"default-org","members":[{"userId":"66fdb875","roleIds":[1,0]}],"createdAt":"2024-05-29T08:07:20.432363793Z"}
{"data":[{"id":"166322a3","name":"default-org","projectCount":0}]}
Creating project default-project ...
{"id":"05bb4710","name":"default-project","members":[{"userId":"66fdb875","roleIds":[1]}],"createdAt":"2024-05-29T08:07:20.505576111Z"}
```
- 执行 `./emqx_sdv_platform_ctl create-org-project` 创建用户名密码（需与上面的保持一致）
```bash
Please input email: sdv@emqx.io
Please input password:
Please input password again:
Creating admin sdv@emqx.io ...
Getting access token ...
{"tokenType":"Bearer","accessToken":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJFTVFYIEVDUCIsInN1YiI6IjZhNzc1MzgyIiwiZXhwIjoxNzE2OTg1NDY3LCJuYmYiOjE3MTY5NzEwNjcsImlhdCI6MTcxNjk3MTA2NywianRpIjoiZmI3MjJiMzEtYjllNC00MGQ1LWIyNzYtZjI4MmNkYTQ4YWI5IiwidHlwIjoiQmVhcmVyIn0.CSzCDp3MQjQdJUBvOMdBuwxscaiItivf9AUy4pk_ZG-Sfl5wUa3NXbZ_ffmp6FrqRrv1LEytxiMEcObX-GlG6dyUV-mm5x7DIj8Iondx2swpbBGZGV_mdjEqFUDztwkiBP8F2J6NjOdJykcffqQEjJxj9OjyOBwUzDf-R_vTTLEmrVfcNt5yPC09mMEgv652mi3FvJt3Eqb43XE8viam7BSREF-zG-bEe7ftBQtBtbTJOlIkzdVvzX6dRXUT74quSJm93BgOGFlm4iJHPZzVgfojMXG7g7IMLC0ip8LivqyvxXMM59ramFYXz-zRceTn-2xPLUcX2clKxYYG8JwOEg","expiresIn":14400,"refreshToken":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJFTVFYIEVDUCIsInN1YiI6IjZhNzc1MzgyIiwiZXhwIjoxNzE3MDU3NDY3LCJuYmYiOjE3MTY5NzEwNjcsImlhdCI6MTcxNjk3MTA2NywianRpIjoiZTcwODE2YTctZWZmZi00NGIwLWE5YTYtOWJlNzViZGIwMzBhIiwidHlwIjoiUmVmcmVzaCJ9.JhUFuEOB0CmO7NcuPo2RuslTSgIm5oWu-9hJa4Fl4paE9OB6yFCaXOLermlZrX5ADbQRlbu1b9HRuAzhk9GhteTkQuc1FAJhnZagf4MdjyDlPMFAPT53AcAhSyMVixuvJJgo4ptoCpfEk1O8ns-arFNvlvENpgYN74hA-GnUsR4F4ta289TfLmnU5fhVg9aBjOn2FWpF2AyCxLf8Ko3M-ckKZXMtGrQUYDpzIbWosk-4W1yVPOjjRcMYHy3Sw16iTe3hZq_wtWXnXrdHzfxXVnAY_VmJxho3VGPVg3_gQfhTs3HO6AIGOvU78cl8BkfMWGBTAdyHPDMmKY-4kV_HeA","refreshExpiresIn":86400,"admin":true}
{"userId":"6a775382","name":"admin","email":"sdv@emqx.io","mobile":"","admin":true}
Creating org default-org ...
{"id":"6db26171","name":"default-org","members":[{"userId":"6a775382","roleIds":[1,0]}],"createdAt":"2024-05-29T08:24:27.615257215Z"}
{"data":[{"id":"6db26171","name":"default-org","projectCount":0}]}
Creating project default-project ...
{"id":"f0b4a050","name":"default-project","members":[{"userId":"6a775382","roleIds":[1]}],"createdAt":"2024-05-29T08:24:27.690085198Z"}
```
- 查看容器是否全部启动，完全启动结果如下：
```bash
$ docker ps -a | grep sdv
e8f92c03806b   emqx/sdv-platform:latest            "/ecp/sdv-platform"      14 minutes ago   Up 14 minutes                   0.0.0.0:8082->8082/tcp, :::8082->8082/tcp                                                                                                                                                                                sdv-platform
9b6fd729f71d   prom/prometheus:v2.37.9             "/bin/prometheus --c…"   14 minutes ago   Restarting (2) 35 seconds ago                                                                                                                                                                                                                            sdv-prometheus
1e6d5195f013   mysql:8.0.18                        "docker-entrypoint.s…"   14 minutes ago   Up 14 minutes                   33060/tcp, 0.0.0.0:3307->3306/tcp, :::3307->3306/tcp                                                                                                                                                                     sdv-db
ad76787c55b0   prom/pushgateway:v1.6.2             "/bin/pushgateway"       14 minutes ago   Up 14 minutes                   0.0.0.0:31901->9091/tcp, :::31901->9091/tcp                                                                                                                                                                              sdv-pushgateway

```

## 车端部署 sdv-flow
sdv-flow 提供多种安装方式，用户可在 安装 中查看详细的安装方式。
下载后下面是解压后文件的默认结构，下面的被标注出来的几部分是我们需要关注的目录。
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
首先需要修改 nanomq 配置，将 `bridges.mqtt.emqx1` 的 server 地址修改为执行 `./emqx_sdv_platform_ctl configure` 所配置的地址。
```conf
bridges.mqtt.emqx1 {
        server = "mqtt-tcp://192.168.21.19:1883"
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
可在 log 目录发现有日志生成，查看日志，发现 sdv-flow，nanomq，kuiper 都已成功运行。
```bash
$ ./bin/sdv-flow run
time="2024-05-29T17:41:36+08:00" level=info msg="load log config:" file="run/sdv-flow.go:72" func=sdv-flow/run.Run
time="2024-05-29T17:41:36+08:00" level=info msg="load log config: {Mode:console Level:info File:log/sdv-flow.log MaxSize:20 MaxBackups:5 ListenAddr: Syslog:{Enable:false Priority: Netword: RemoteAddr: Tag:}}" file="run/sdv-flow.go:76" func=sdv-flow/run.Run
time="2024-05-29T17:41:36+08:00" level=info msg="get pid of sdv-flow: 3306071" file="run/sdv-flow.go:95" func=sdv-flow/run.Run
time="2024-05-29T17:41:36+08:00" level=info msg="Init plugin 'getvin'.\n" file="getvin/getvin.go:79" func="plugins/getvin.(*PluginGetvin).Init"
time="2024-05-29T17:41:36+08:00" level=info msg="Running plugin 'getvin'.\n" file="getvin/getvin.go:88" func="plugins/getvin.(*PluginGetvin).Run"
time="2024-05-29T17:41:36+08:00" level=info msg="Get vin:%!(EXTRA string=ubuntu)" file="getvin/getvin.go:111" func="plugins/getvin.(*PluginGetvin).Run"
time="2024-05-29T17:41:36+08:00" level=info msg="Listening on tcp://127.0.0.1:40899\n" file="getvin/getvin.go:50" func=plugins/getvin.vinServer
time="2024-05-29T17:41:36+08:00" level=info msg="trigger child process with manage: true" file="run/sdv-flow.go:110" func=sdv-flow/run.Run
time="2024-05-29T17:41:36+08:00" level=info msg="trigger eKuiper with command: GOTRACEBACK=crash KUIPER__BASIC__SYSLOG__ENABLE=true KUIPER__BASIC__SYSLOG__LEVEL=debug KUIPER__BASIC__SYSLOG__NETWORK=udp4 KUIPER__BASIC__SYSLOG__address=localhost:10514  KUIPER__BASIC__RESTIP=127.0.0.1 KUIPER__BASIC__PROMETHEUS=true KUIPER__BASIC__PROMETHEUSPORT=9081 /home/ubuntu/workspace/test/sdv-flow-0.9.0-beta.3-linux-arm64/software/ekuiper/bin/kuiperd -loadFileType absolute -etc /home/ubuntu/workspace/test/sdv-flow-0.9.0-beta.3-linux-arm64/software/ekuiper/etc -data /home/ubuntu/workspace/test/sdv-flow-0.9.0-beta.3-linux-arm64/data/ekuiper/data -log /home/ubuntu/workspace/test/sdv-flow-0.9.0-beta.3-linux-arm64/software/ekuiper/log -plugins /home/ubuntu/workspace/test/sdv-flow-0.9.0-beta.3-linux-arm64/data/ekuiper/plugins\n" file="process_control/process.go:73" func=internal/process_control.get_ekuiper_Process
time="2024-05-29T17:41:36+08:00" level=info msg="trigger nanomq with command: rm -f /tmp/nanomq/nanomq.pid && cd /home/ubuntu/workspace/test/sdv-flow-0.9.0-beta.3-linux-arm64/software/nanomq/ &&NANOMQ_VIN=ubuntu ./nanomq start --conf ./etc/nanomq.conf\n\n" file="process_control/process.go:85" func=internal/process_control.get_nanomq_Process
time="2024-05-29T17:41:38+08:00" level=info msg="[Agent] is starting, vin: ubuntu" file="agent/init.go:18" func=sdv-flow/agent.Init
time="2024-05-29T17:41:38+08:00" level=error msg="path not exist" file="agent/clean_parquet.go:17" func=sdv-flow/agent.CleanParquetFile
time="2024-05-29T17:41:38+08:00" level=info msg="[MQTT]connected to tcp://127.0.0.1:1883" file="agent/mqtt.go:70" func=sdv-flow/agent.onMQTTConnected
time="2024-05-29T17:41:38+08:00" level=info msg="[Agent]heartbeat is enabled ,interval 15" file="agent/init.go:66" func=sdv-flow/agent.Init
time="2024-05-29T17:41:38+08:00" level=info msg="MQTT Tunnel Proxy Agent connected to tcp://127.0.0.1:1883" file="mqtt/proxy.go:88" func=ecp-tunnel/mqtt.SubTunnelTopic
time="2024-05-29T17:41:38+08:00" level=info msg="MQTT Tunnel Proxy Subscribed topic agent/ubuntu/proxy/request/+" file="mqtt/proxy.go:116" func=ecp-tunnel/mqtt.SubTunnelTopic.func1
```
此时可以订阅 `ecp/#` 获取边缘端状态![](_assets/sdv-flow-heartbeat.png)，如图所示，

也可以在 software/nanomq/log 和 software/ekuiper/log 目录查看 nanomq 和 ekuiper 对应组件的日志。此时的 nanomq 默认已开启 mq stream 落盘和文件上传功能。

### 落盘功能
查看 nanomq 配置 software/nanomq/etc/nanomq.conf 
```conf
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
我们可以发送 mqtt 消息到 canudp 主题。根据上面的配置可以观测到每一千条消息，会触发一次落盘，具体根据数据内容的大小，可能会生成多个文件。数据落盘文件在 parquet 目录。持续发消息会有新的文件落盘。生成得非加密的文件可以通过 parquet-tools 查看结果。