---
title: "MQTT"
type: entity
tags: [协议, 物联网, 消息传输, 发布订阅]
sources: [raw/01-articles/20分钟搞定MQTT协议网络搭建及程序实现.txt, raw/01-articles/MQTT协议快速入门.md, raw/01-articles/SubBatch_2026-08-12-09-08-32_TXT/]
last_updated: 2026-08-12
---

## 定义

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输）是一种基于**发布/订阅模式**的**轻量级消息传输协议**，设计用于低带宽、高延迟或不稳定的网络环境，是物联网领域事实上的标准协议。它运行于 TCP/IP 之上，以**中心化 Broker** 为消息中枢，客户端之间通过**主题（Topic）** 解耦通信，而非直接点对点。

## 关键信息

- **版本更迭**：1999 年 v1.0 由 IBM Andy Stanford-Clark 与 Eurotech 的 Arlen Nipper 为石油管道远程监控发明；2010 年 v3.1 开放；2014 年 v3.1.1 成为 OASIS 标准（当前主流）；2019 年 v5.0 成为新标准。
- **三大角色**：**发布者（Publisher）** 产生并发送消息；**代理服务器（Broker）** 居中转发与存储消息；**订阅者（Subscriber）** 按主题接收消息。一个客户端可同时是发布者与订阅者。
- **消息模型**：通信不依赖 IP，而依赖 **Topic（主题，字符串）+ Payload（负载，消息内容）**。发布者向某主题发布消息，订阅同名主题的任意订阅者自动收到——类似广播电台的"频道"。详见 [[发布订阅模型]]。
- **五大优势**：① 跨语言（C/C++/Java/Python/Go 均有官方/社区客户端）② 建立在 TCP/IP 之上安全可靠 ③ [[MQTT服务质量]]（QoS 0/1/2）④ 心跳保活识别假死连接（若在 **1.5×keepAliveInterval** 内无任何报文或 PINGREQ/PINGRESP，连接即被判假死而断开）⑤ 持久会话（离线消息保留，订阅端上线即取）。
- **报文结构**：固定报头（报文类型 + 标志）+ 可变报头（主题、报文标识符）+ 有效载荷，详见 [[MQTT报文结构]]。
- **默认端口**：MQTT over TCP 为 **1883**；EMQX 等 Broker 的 Dashboard 管理页通常为 18083；另有 TLS 8883、WebSocket 8083/8084。
- **公共测试 Broker**：EMQX 官方提供免费公共实例 `broker.emqx.io:1883`（TLS `8883`），无需自建服务器即可联调客户端。
- **生态组件**：常见 Broker 为 [[EMQX]]（大规模分布式）与 Eclipse Mosquitto（轻量单机）；客户端工具为 [[MQTTX]]（图形化）；编程侧用各语言 MQTT 客户端库（C 用 [[Paho]] C 客户端、Python 用 paho-mqtt、Java 用 org.eclipse.paho 等）。

## 关联连接
- [[发布订阅模型]] — 核心工作模型
- [[MQTT报文结构]] — 数据包格式
- [[MQTT服务质量]] — QoS 0/1/2
- [[MQTT搭建流程]] — 从零搭建通信网络的通用流程
- [[EMQX]] / [[MQTTX]] — 常用 Broker 与客户端工具
- [[Paho]] — 常用 C 客户端库
- [[同步异步客户端API]] — 客户端库的同步/异步设计
- [[消息队列]] — 同源的中间件设计思想
- [[摘要-mqtt网络搭建]] / [[摘要-mqtt协议快速入门]] / [[摘要-mqtt协议介绍系列]] / [[摘要-mqtt协议大全-pdf]] / [[摘要-paho-c客户端教程]] — 来源
