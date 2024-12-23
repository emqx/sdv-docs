# SDV-Platform 介绍

sdv-platform 车云协同管理平台是 EMQ 提供对 sdv-flow 进行统一管理的程序，部署在云端，提供标准 REST API 与 sdv-flow 交互。主要功能包括车辆注册、数采规则的编辑与下发、车端数据的实时采集与处理、状态检测与资源占用上报，以及日志文件的获取与上传。通过EMQX，平台支持海量车端数采软件节点的并发接入和数据汇聚，同时保证了数据传输的高可靠性和低延迟。

作为云边协同管理平台，它提供了组件注册、管理监控、规则编辑创建等管理服务，确保了数据采集的灵活性和实时性。同时它还提供用户管理，组织项目管理，方便企业更好的管理控制车端。

sdv-platform 车云协同管理平台的设计考虑了扩展性和安全性，支持私有化部署，具备良好的能力开放集成特性，其提供的面向车端数采基础性管理能力，可以通过API形式开放，供上层车云协同业务管理平台集成。通过这种灵活的数采工具链，用户可以轻松应对各种数据采集和分析需求，从而推动车辆智能化和数据驱动的决策制定。

sdv-flow 和 sdv-platform 之间通过 MQTT 协议进行通信，其中包括，

- 车端通过 `ecp` 开头的 MQTT 主题上报状态
- 云端通过 `agent` 开头的 MQTT 主题下发规则或指令

sdv-flow 启动后，将自动获得 vin 码，代表车端的唯一标识。如果车辆从未注册过，那sdv-flow 启动会不断的通过emqx向云端发申请注册报文，直到云端下发注册成功的响应报文。如果license 不足，将会注册失败。如果注册成功，则后面将不会再注册，如果失败，则车端起他功能均不可用。

如果 sdv-flow 注册成功后，会不断向云端定时发送心跳信息，包括各个服务的运行状态，资源占用情况和指标信息，云端可以通过订阅主题 `ecp/#` 获得车端上报的信息。