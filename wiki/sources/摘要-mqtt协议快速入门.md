---
title: "摘要-mqtt协议快速入门"
type: source
tags: [来源, MQTT, 物联网, 报文]
sources: [raw/01-articles/MQTT协议快速入门.md]
last_updated: 2026-08-12
---

## 核心摘要
- 本资料是《MQTT 协议快速入门》技术博客剪藏，从协议定义、版本更迭、应用场景、报文结构到 Broker 安装与 Java 代码实践，覆盖面广。
- 定义：MQTT 是**基于发布/订阅模式的轻量级消息传输协议**，设计用于低带宽、高延迟或不稳定的网络环境，广泛应用于物联网。
- 版本更迭：1999 年 v1.0（IBM Andy Stanford-Clark 与 Eurotech Arlen Nipper，石油管道远程监控发明）→ 2010 年 v3.1 开放 → 2014 年 v3.1.1 成为 OASIS 标准（当前主流）→ 2019 年 v5.0（OASIS 新标准）。
- 应用场景：智能家居、车联网、工业物联网、远程监控控制、消息通知、资源监控管理、数据采集分析。
- 五大优势：① 跨语言（C/C++/Java/Python/Go 均有客户端）② 建立在 TCP/IP 之上安全可靠 ③ 三种服务质量 QoS 0/1/2 ④ 心跳保活识别假死连接 ⑤ 持久会话保留离线消息。
- 报文结构：**固定报头**（所有包都有，byte1 高 4 位=报文类型、低 4 位=DUP/QoS/RETAIN 标志）+ **可变报头**（主题名、报文标识符）+ **有效载荷**（消息内容）；14 种报文类型（CONNECT/CONNACK/PUBLISH/PUBACK/PUBREC/PUBREL/PUBCOMP/SUBSCRIBE/SUBACK/UNSUBSCRIBE/UNSUBACK/PINGREQ/PINGRESP/DISCONNECT）。
- Broker 对比：**Mosquitto**（开源轻量、支持 3.1/3.1.1/5.0、不支持集群）；**EMQX**（开源大规模分布式、单台并发可上亿、每秒百万级消息）。
- EMQX Windows 安装八步：下载 zip → 解压 → 管理员命令行进 bin → `emqx.cmd install` → 服务列表启动 → `emqx.cmd console` 确认 → 浏览器访问 `127.0.0.1:18083`（默认 admin/public）；开发用"服务+console"，正式环境直接 `emqx start`。
- 客户端工具：EMQX 官网自带 **MQTTX**（跨平台桌面客户端）。Java 侧用 `org.eclipse.paho.client.mqttv3` 依赖实现订阅者/发布者。

## 关联连接
- [[MQTT]] — 该资料讲解的协议本体
- [[EMQX]] / [[MQTTX]] — 该资料的 Broker 与客户端工具
- [[MQTT报文结构]] — 固定报头/可变报头/载荷
- [[MQTT服务质量]] — QoS 0/1/2
- [[MQTT搭建流程]] — 该资料演示的搭建过程抽象
