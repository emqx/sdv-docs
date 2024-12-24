# Geely 交付版本历史

# 2024-12-23
- 修改默认selinux权限文件

# 2024-12-19-1
- kuiper 修改了周期采集规则的busid
- 修改了kuiper 和nanomq的监听端口
- 修改了nanomq 桥接订阅主题，不再使用通配符。

# 2024-12-16-1
- add new vin logic (give up trying in 10s + retry once even file exist) + more log print 
- fix dataswitch create failed

# 2024-12-12-1
- Retry fetch VIN actively if no cb is triggered
- check if VIN is valid

# 2024-12-06-1
- 更新ekuiper 的数据处理规则

# 2024-12-05-1
- 增加bigdata文件夹创建权限
- 更新nanomq的异常捕获
- 增加parquet文件落盘最大数量到3000
- 启用TLS连接

# 2024-12-04-1
- 删除若干权限
- 更新ekuiper

# 2024-12-02-1
- 为 RVDC 的问题修改 geely_api proxy的代理订阅主题，避免他们要改。
- 更新nanomq 到 0.22.10-beta.23，与主线同步
- 更新 kuiper 到最新版本
- 新增若干权限

# 2024-11-28-1
- 修复 geely_api 的代理订阅会因为客户端ID竞争导致的崩溃问题
- 更新nanomq 到 0.22.10-beta.22，对时间跳变造成的时间戳重复影响落盘问题做了特殊定制处理
- 更新nanomq 的 quic桥接，现在支持多流

# 2024-11-27-1
- 增加 dnsproxyd_socket vendor_toolbox_exec toolbox_exec 的权限

# 2024-11-26-1
- 配置默认关闭软开关功能 （调试目的）
- 默认sdv-flow log最大保存数量为500个
- 默认nanomq的日志等级为info （调试目的）

# 2024-11-25-1
- sepolicy
  - 新增netd sepolicy
  allow netd sdv-flow:fd use;
  allow netd sdv-flow:tcp_socket { getopt read setopt write };

  - 新增node权限和kernel权限
  allow sdv-flow kernel:file getattr;
  allow sdv-flow node:tcp_socket node_bind;

# 2024-11-23-5
- nanomq
  移除文件传输客户端
- sdv-flow
  版本更新，修复json文件路径异常问题
  在yaml中打开ft开关，使得geely_api的文件传输客户端启动

# 2024-11-23-4
- libmbedtls相关so恢复

# 2024-11-23-3
- Android.bp
  移除libmbedtls相关so 移除shared_libs

# 2024-11-23-2
- Android.bp
  所有library_shared都改为check_elf_files: false

# 2024-11-23-1
- Android.bp
  调整target和shared_lib的顺序

# 2024-11-22-8
- Android.bp
  尝试调整so的顺序，ci报错说找不到libmbedtls
  Android.bp module "libxxxx" variant "android_vendor.30_arm_armv8-a_static": module "libmbedcrypto"missing output file

# 2024-11-22-7
- Android.bp
  修复一处语法错误影响libmbedtls

# 2024-11-22-6
- nanomq
  更新桥接服务器域名为生产环境

# 2024-11-22-5
- geely_api
  修复batt可能卡住无故障灯数据上传的问题
- sdv-flow
  支持在sdv-flow.yaml中配置软开关是否启用以及vin码检查不合法关闭桥接的功能
  配置文件增加软开关是否启用以及vin码检查不合法关闭桥接的功能

# 2024-11-22-4
- sepolicy
  根据ci增加prebuilt_library的依赖so

# 2024-11-22-3
- sepolicy
  恢复nanomq kuiperd geely_api的te文件，否则找不到对象

# 2024-11-22-2
- sepolicy
  修改sepolicy角色，给sdv-flow增加三个可运行二进制增加no_execute_trans

# 2024-11-22-1
- device.mk
  增加拷贝so的动作
- geely_api
  移除清空ssl文件夹的异常行为

# 2024-11-21-2
- geely_api
  bug修复
  增加syslog日志输出
  增加vin码校验，如果vin码不合法则给全0的vin码

- sdv-flow
  sdv-flow.yaml中增加geely_api syslog的日志文件socket参数
  1.修复了 signal handler 之前 kill 不掉问题
  2.flow 收到 sub 读取到值变化才会重启
  3.重启会检查删除 周期上传目录是否有 zst 文件
  4.restart.json 加了然是判断，超过一分钟的不会文件会被删掉

- nanomq
  修复nanomq.conf中batt topic name不一致的问题
  nanomq.conf桥接改为域名连接
  nanomq.conf修复${VIN}占位符的问题
  同步最新的nanomq.conf

# 2024-11-21-1
- geely_api
  更新

# 2024-11-20-3
- sepolicy
  移除一些看起来不需要的权限

# 2024-11-20-2
- sepolicy
  移除dac overide,sys_ptrace,shell_data_file:dir{search},toolbox_exec:file{...}权限

# 2024-11-20-1
- sdv-flow
  - 修复一些异常行为
  - 支持全局LD_LIBRARY_PATH
- nanomq
  - support specify qos in file transfer cmd
  - new pardon topics
  - memleak fix
  - 替换成动态链接版本，支持域名解析
- geely_api
  - 进程合并后出现偶然卡死问题解决
  - 优化代码
- lib
  - add library dir
- Android.bp
  - add cc_prebuilt_library_shared

# 2024-11-19-1
- sdv-flow
  - 代码统一，和linux_android版本保持一致
  - 移除对shell/sh/find进程的调用逻辑，改为使用libc接口进行处理
  - 增加对文件的读写权限检查
- geely_api
  - 合并geely_api为一个进程
  - 增加kuiper suback响应
  - signal主题的消息会缓存10条，直到kuiper订阅成功
- nanomq
  - 增加signal 5 的处理逻辑，收到signal 5后对内存中的消息进行落盘
  - 文件传输客户端支持qos
  - 故障灯主题修改
- kuiper
  - SPI 解析: 支持配置 parseNoneUpdate，默认为 false，即不解析 update bit = 0的Ratelimit 聚合时间戳以聚合时间为准 (保证周期采集每行时间间隔基本接近)
  - 规则添加 notivSub 选项，规则的 source 订阅成功后会定时每秒本地 broker 的ctr/subready topic 发送订阅成功的流名字o更改信号灯诊断规则，配置 notifySub 选项
  - 处理 sigtrap (与 sigterm 相同)

# 2024-11-18-1
- sepolicy
  default_vendor不允许对file的读写操作

# 2024-11-17-1
- sepolicy
  移除sepolicy里面sdv-flow的shell权限

# 2024-11-16-1
- sepolicy
  - 新增一些未识别到的selinux权限

# 2024-11-15-1
- nanomq
  修复nanomq启动后概率异常退出的问题
  nanomq配置中spi数据落盘主题由canudp改为canspi
- kuiperd
  - CAN 解析分开处理int和float，支持 big number (超过int64 的整数)
  - 规则:
    - SPI 流的 topic 由 canudp 改成 canspi
    - MQTT sink 都设置 qos 1
    - 更新诊断规则6
  - 打包:
    - JSON 文件 minify
    - Gee 版本只打包 gee 相关的包0
- spi_proxy
  - spi发送的主题由canudp改为canspi

# 2024-11-14-1
- nanomq
  增加断网重传的优化，包含配置项的修改
  优化断网重传的逻辑，增加重传的次数和间隔时间的配置
- sdv-flow
  sdv-flow不再检测到pid.json存在就异常退出

# 2024-11-13-3
- kuiper (new DBC and remove warning log of invalid input data)

# 2024-11-13-2
- geely_api (signal will wait 20s to transfer for kuiper rules ready)

# 2024-11-13-1
- kuiper
  故障灯算法初始只上传有1的信号组(即默认信号为0，有1则表示有变化)
  更新 dbc json 和周期采集
  删除 vin 等用户属性配置输出的 log
  Fix: 事件和诊断部分信号的 can id 改为3位,例如 0x053。Fix: 修复 MQTT 连接状态获取失败可能导致规则停止的问题

# 2024-11-11-4
- New kuiper version(more new signal rules)

# 2024-11-11-3
- Update kuiperd binary(new signal rules)

# 2024-11-11-1
- Update nanomq binary
- Updata sdv-flow(New dbc path for ekuiper)

# 2024-11-09-1
- Update nanomq/kuiper/sdv-flow binary
- New etc.zip for nanomq/kuiper/sdv-flow
  - Modify nanomq.conf
  - Update kuiper data directory
- Update sdv-flow.yaml
  - Start proxy with LD_LIBRARY_PATH
  - disable aes in yaml

# 2024-11-08-2
- Update nanomq(for raw stream bugfix)
- Update nanomq.conf(more configuration is added)
- Add geely_api binary
- Add libsecretkey binary
- Add spi_proxy binary
- Update sdv-flow.yaml(Add geely_api and libsecretkey and spi_proxy auto start)
- Update sdv-flow(Use sh to start geely_api and libsecretkey and spi_proxy)

# 2024-11-08-1
- Update sdv-flow(Write default value to vin when vin file is empty)
- Update new etc.zip(new nanomq.conf for multiple stream and parquet path)
- Update sdv-flow.te
```
allow sdv-flow cgroup_bpf:dir search;
allow sdv-flow self:netlink_route_socket { getattr nlmsg_read };
allow sdv-flow self:tcp_socket shutdown;
```

# 2024-11-07-2
- Update sepolicy for sdv-flow/nanomq/ekuiper(allow sdv-flow to access /bigdata)

# 2024-11-07-1
- Remove NANOMQ_PID_FILE in sdv-flow.rc
- Update sdv-flow binary(for NANOMQ_PID_FILE)

# 2024-11-06-1
- Update sdv-flow/nanomq/ekuiper
- Support unzip /vendor/etc/emq/etc.zip to /bigdata
- Add version file to control sdv-flow version
- Remove library and geely_api binary(will add soon)
