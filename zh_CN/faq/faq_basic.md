# FAQ 常见问题解答

## SDV-Flow 是什么

sdv-flow 是 EMQ 基于开源软件和车载标准协议提供的车内协议接入及实时数据处理程序，sdv-flow 读取车内 CAN、SOME-IP、DDS等协议数据，根据预定义 SQL 规则或通过 sdv-platform 实时下发 SQL 规则，将车内协议产生数据进行处理并以指定方式存储或通过网络转发 。

## SDV-Platform 是什么

sdv-platform 是 EMQ 提供对 sdv-flow 进行统一管理程序，提供标准 REST API 与 sdv-flow 交互 。

## SDV-Flow 是否支持 AI、ML 集成分析

sdv-flow 支持用户自定义函数扩展和 AI 算法集成，提供智能数据分析能力。

## 云端如何读取车端数据

云端支持从 emqx 直接通过 mqtt 协议订阅获取车端实时数据，也支持通过 emqx 规则引擎将数据转发至 kafka、S3、时序数据库、关系数据库等存储中读取数据

## 软件运行环境支持哪些系统？

sdv-platform 支持基于 Linux 内核的 ARM64 和 x86_64 操作系统。
sdv-flow 支持基于 Linux 内核的 ARM64 和 x86_64 操作系统（包括不限于 Android、Raspbian）

sdv-flow 和 sdv-platform 均不支持 Windows 操作系统直接运行，支持以下几种安装方式在 Windows 上体验调试:

- 使用虚拟化软件安装相应的 Linux 系统
- 使用 Docker 容器，以容器的方式安装和运行 


## 是否支持 Docker 容器化部署？

 sdv-flow 支持 Docker 容器化部署
 sdv-platform 支持 Docker 容器化部署

## 软件安装需要的硬件配置

sdv-flow 启动需要约 17MB 基础内存，受每秒消息量及规则复杂度影响消耗更多CPU及内存
