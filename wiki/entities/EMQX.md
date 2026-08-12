---
title: "EMQX"
type: entity
tags: [MQTT, 服务器, Broker, 开源软件]
sources: [raw/01-articles/MQTT协议快速入门.md, raw/01-articles/SubBatch_2026-08-12-09-08-32_TXT/]
last_updated: 2026-08-12
---

## 定义

EMQX 是一款开源的**大规模分布式 MQTT 消息代理服务器（Broker）**，基于 Erlang/OTP 构建，支持 MQTT 3.1 / 3.1.1 / 5.0 版本。它是 MQTT 通信网络中的"消息中枢"：所有发布者与订阅者都连接到它，由它完成消息的路由、转发、存储与鉴权。

## 关键信息

- **性能规模**：单台并发连接数可高达一亿，每秒可处理百万级消息，面向工业物联网、车联网、车路协同等大规模接入场景。
- **可视化控制台**：内置 Dashboard，默认地址 `http://<IP>:18083`，默认账号 `admin` / 密码 `public`（首次登录引导改密）。可在页面上实时查看客户端连接数、消息收发频率、主题分布与在线状态，是验证搭建是否成功的核心手段。
- **多协议监听端口**：MQTT/TCP `1883`、MQTT/TLS `8883`、WebSocket `8083`、WebSocket/TLS `8084`、Dashboard HTTP `18083`。
- **免费公共测试实例**：官方提供公共 Broker `broker.emqx.io:1883`（TLS `8883`），无需自建即可联调客户端，是 MQTT 教程与快速验证的常用后端。
- **集群高可用**：EMQX Cluster 分布式架构支撑百万级连接；客户端所连节点故障时可**自动重连**集群内其它可用节点，`cleansession=0` 可保留会话状态以最小化停机影响。
- **安全能力**：原生支持 TLS/SSL 多端口、客户端证书认证（Client Certificate Authentication）与外部安全后端集成，满足生产环境的信任层要求。
- **跨平台安装**：官网提供 Windows / Linux（Ubuntu、Debian、CentOS）/ macOS / Docker 等安装包；有中文官网与文档，国内使用广泛。
- **Windows 安装要点**：下载 zip → 解压 → 以管理员身份打开命令行进入 `bin` 目录 → `emqx.cmd install` 注册服务 → 在 Windows 服务列表启动 → `emqx.cmd console` 启动监控 → 浏览器访问 Dashboard。**开发环境**用"服务 + console"两步启动；**正式环境**（如 Linux 服务器）直接用 `emqx start` 一条命令同时拉起服务与监控。
- **同类对比**：Eclipse Mosquitto 同为开源 Broker，更轻量、适用低功耗单板到完整服务器，但**不支持集群**；EMQX 则主打大规模分布式与高并发。

## 关联连接
- [[MQTT]] — 其承载的协议本体
- [[MQTTX]] — 官方配套客户端工具
- [[Paho]] — 官方教程搭配的 C 客户端库
- [[MQTT搭建流程]] — 其安装/启动属于流程中的"部署 Broker"阶段
- [[摘要-mqtt协议快速入门]] / [[摘要-mqtt协议介绍系列]] / [[摘要-paho-c客户端教程]] — 来源
