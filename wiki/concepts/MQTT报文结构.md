---
title: "MQTT报文结构"
type: concept
tags: [MQTT, 报文, 协议, 二进制]
sources: [raw/01-articles/MQTT协议快速入门.md, raw/01-articles/SubBatch_2026-08-12-09-08-32_TXT/]
last_updated: 2026-08-12
---

## 定义

MQTT 报文（数据包）由三部分构成：**固定报头（Fixed header）** + **可变报头（Variable header）** + **有效载荷（Payload）**。固定报头存在于所有数据包中，标识报文类型与关键标志；可变报头与有效载荷根据报文类型决定是否存在及具体内容。

## 关键信息

- **固定报头（byte1 + 剩余长度）**：
  - **byte1 高 4 位**：报文类型。0–15 取值对应 14 种有效类型 + 2 个保留值：
    | 值 | 报文 | 方向 | 含义 |
    |---|---|---|---|
    | 1 | CONNECT | C→S | 客户端请求连接 |
    | 2 | CONNACK | S→C | 连接确认 |
    | 3 | PUBLISH | 双向 | 发布消息 |
    | 4 | PUBACK | 双向 | QoS1 发布确认 |
    | 5 | PUBREC | 双向 | QoS2 发布收到（第一步） |
    | 6 | PUBREL | 双向 | QoS2 发布释放（第二步） |
    | 7 | PUBCOMP | 双向 | QoS2 发布完成（第三步） |
    | 8 | SUBSCRIBE | C→S | 订阅请求 |
    | 9 | SUBACK | S→C | 订阅确认 |
    | 10 | UNSUBSCRIBE | C→S | 取消订阅 |
    | 11 | UNSUBACK | S→C | 取消订阅确认 |
    | 12 | PINGREQ | C→S | 心跳请求 |
    | 13 | PINGRESP | S→C | 心跳应答 |
    | 14 | DISCONNECT | C→S | 断开连接 |
  - **byte1 低 4 位**：仅当类型为 PUBLISH(3) 时有意义，为 **DUP / QoS(2bit) / RETAIN** 标志：DUP 表示是否重发，QoS 表示[[MQTT服务质量]]等级，RETAIN 表示是否要求 Broker 保留该主题的最后一条消息。
  - **剩余长度（Remaining Length）**：可变报头 + 有效载荷的字节数，可变长编码（1–4 字节）。
- **可变报头**：随报文类型变化，典型包含**主题名（Topic Name）**与**报文标识符（Packet Identifier，用于 QoS>0 的确认匹配）**。
- **有效载荷**：消息的实际内容（Payload），如业务数据。
- **分层视角**：MQTT 报文的"类型 + 标志"与业务负载分离，本质是[[协议分层]]思想在单包格式上的体现——固定报头是"信封"，有效载荷是"信纸"。

## 关联连接
- [[MQTT]] — 其报文格式规范
- [[MQTT服务质量]] — QoS 标志位与确认流程
- [[协议分层]] — 报文封装背后的分层思想
- [[摘要-mqtt协议快速入门]] / [[摘要-mqtt协议介绍系列]] — 来源
