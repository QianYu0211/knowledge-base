---
title: "摘要-mqtt协议介绍系列"
type: source
tags: [来源, MQTT, 物联网, 视频课程]
sources: [raw/01-articles/SubBatch_2026-08-12-09-08-32_TXT/]
last_updated: 2026-08-12
---

## 核心摘要
- 本资料是「MQTT 协议介绍」20 集视频课程的转录合集（P1–P20），从协议概念到 EMQX 安装、MQTTX 使用、Java 代码实现，是完整的快速入门体系。
- P1–P4 协议概念：物联网"万物互联"时代 MQTT 应用极广；基于发布/订阅的轻量级协议，服务硬件/网络资源匮乏场景；版本更迭、七大应用场景、五大优势（跨语言、TCP/IP 可靠、QoS 0/1/2、心跳保活、持久会话）。
- P5–P7 报文：报文三部分结构，固定报头 byte1 高 4 位为报文类型、低 4 位为 DUP/QoS/RETAIN 标志（仅 PUBLISH 时低 4 位有意义）。
- P8 工作原理：两点通信必经由**中间代理（Broker）** 转发；用收音机/广播电台频道比喻 Topic——订阅者"调频"到某主题即可收到该主题消息。
- P9–P11 Broker：介绍 Mosquitto（Eclipse 系）与 EMQX；Windows 上 EMQX 下载（官网选平台版本）→ 解压 → 管理员 CMD 进 bin → `emqx.cmd install` → 服务启动 → `emqx.cmd console` 起监控 → 浏览器 `127.0.0.1:18083`（admin/public）进入图形控制台；强调"服务 + 监控要起两次"、正式环境用 `emqx start`。
- P12–P14 客户端工具：MQTTX 是 EMQX 官方跨平台 MQTT 5.0 桌面客户端，类比 Navicat 之于 MySQL——不写代码即可向 Broker 发消息/订阅消息做连通性测试。
- P15–P19 代码实践：Java 使用 `org.eclipse.paho.client.mqttv3`；**先写订阅者再写发布者**（收音机比喻：订阅者必须时刻保持存活，否则错过广播）；订阅者=创建 client + 连接配置（cleanSession 等）+ setCallback（connectionLost/messageArrived/deliveryComplete）+ subscribe(topic)；发布者=创建 client + connect + 构造 MqttMessage + publish(topic, msg) + 发布完可断开；默认端口 1883。
- P20 总结：MQTT 就是用于物联网的数据传输协议，工作原理与消息队列（RocketMQ/Kafka）相通；学习路径 = 装 Broker → 写代码，各语言流程本质相同。

## 关联连接
- [[MQTT]] — 该系列讲解的协议本体
- [[EMQX]] / [[MQTTX]] — 该系列使用的 Broker 与客户端工具
- [[MQTT搭建流程]] — 该系列安装/编码流程的抽象
- [[MQTT报文结构]] — P5–P7 详解
- [[MQTT服务质量]] — QoS 三等级
