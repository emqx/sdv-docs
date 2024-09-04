# 配置管理

## 配置文件
sdv-flow 日志由以下几部分部分构成:

- [数据总线配置](./nanomq.md)：默认配置文件位于 `software/nanomq/etc/nanomq.conf`
- [协议转换配置](./nanolink.md)：默认配置文件位于 `software/nanolink/etc/nanolink.conf`
- [流处理引擎配置](./ekuiper.md)：默认配置文件位于 `software/ekuiper/etc`
- [边缘协同代理配置](./sdv-flow.md)：默认配置文件位于 `etc/sdv-flow.conf`

sdv-platform 配置：
- [云管理平台配置](./platform.md)


## 命令行
sdv-flow 的命令行位于 /bin/sdv-flow，它提供了以下的常用选项：

```shell
-c, --config 配置文件路径（默认为 "etc/sdv-flow.yaml"）
-h, --help 运行帮助
-m, --manage 管理 eKuiper 和 NanoMQ 的生命周期（默认为 true）
```
### `run` 命令
`run` 命令用于在控制台上运行 sdv-flow。该命令将 sdv-flow 作为一个进程启动，并在终端中显示其输出。

例如：

```sh
./bin/sdv-flow run -c etc/sdv-flow.yaml -m false
```
该命令将 sdv-flow 作为进程启动，并在终端中显示其输出。sdv-flow 不会管理 NanoMQ 和 eKuiper 的生命周期。

sdv-flow支持在启动时指定注册的组织和项目，如果不指定则会注册到默认的组织项目

```
./bin/sdv-flow run  -o=248a6080 -p=49e7b280 
```

### `start` 命令
`start` 命令用于在守护进程模式下启动 sdv-flow，该命令将 sdv-flow 作为守护进程启动并在后台运行。

例如

```sh
./bin/sdv-flow start -c etc/sdv-flow.yaml -m false
```
该命令将 sdv-flow 作为守护进程启动，并在后台运行。sdv-flow 不会管理 NanoMQ 和 eKuiper 的生命周期，也不会开启权限验证。

在启动时指定注册的组织和项目

```
./bin/sdv-flow start -o=248a6080 -p=49e7b280 
```

### `stop` 命令
`stop` 命令用于停止运行 sdv-flow。该命令将杀死 sdv-flow 进程。

```sh
./bin/sdv-flow stop
```
### `install` 命令
`install` 命令用于在 /etc/systemd/system path 中注册 sdv-flow 服务配置文件。

```sh
./bin/sdv-flow install
```
### `uninstall` 命令
`uninstall` 命令用于在 /etc/systemd/system path 中取消注册 sdv-flow 服务配置文件。

```sh
./bin/sdv-flow uninstall
```