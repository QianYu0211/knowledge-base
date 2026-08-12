---
title: "Paho"
type: entity
tags: [MQTT, 客户端库, 开源软件, C语言]
sources: [raw/01-articles/Mastering MQTT in C A Step-by-Step Tutorial Using the Paho C Client.md]
last_updated: 2026-08-12
---

## 定义

Eclipse Paho 是 Eclipse 基金会旗下的**开源 MQTT 客户端库系列**，为 C、Python、Java、Go 等多种语言提供工业标准实现。其中 **Paho C 客户端（`eclipse-paho/paho.mqtt.c`）** 是 C 语言 MQTT 开发的事实标准，广泛用于嵌入式系统与高性能、低延迟场景。

## 关键信息

- **安装**（Unix-like）：`git clone https://github.com/eclipse-paho/paho.mqtt.c.git` → `make` → `sudo make install`，安装头文件（`MQTTClient.h` 等）与链接库。
- **两套 API**：
  - **同步版 `MQTTClient.h`**：阻塞调用、代码线性简单；`connect()` 阻塞到连接完成；适合教学与简单场景；链接库 `paho-mqtt3c`。
  - **异步版 `MQTTAsync.h`**：非阻塞 + 回调事件通知（连接建立、消息到达），适合高并发与单线程嵌入式；详见 [[同步异步客户端API]]。
- **同步 API 核心函数**：`MQTTClient_create`（建客户端）→ `MQTTClient_setCallbacks`（绑消息回调 `messageArrived`）→ `MQTTClient_connect`（连 Broker、检查返回码）→ `MQTTClient_subscribe`（订阅主题）→ `MQTTClient_publishMessage` + `MQTTClient_waitForCompletion`（发布并阻塞等确认）→ `MQTTClient_disconnect`/`MQTTClient_destroy`（清理）。
- **回调机制**：`messageArrived(void *ctx, char *topicName, int topicLen, MQTTClient_message *msg)` 每收到一条消息触发一次，对应 [[发布订阅模型]] 的订阅者消费模型。
- **TLS 变体**：库分 `paho-mqtt3c`（明文）、`paho-mqtt3cs`（TLS 同步）、`paho-mqtt3as`（TLS 异步）；TLS 用 `MQTTClient_SSLOptions` 并将地址改为 `ssl://<broker>:8883`。
- **MQTT 5.0**：完整支持（Session Expiry、User Properties），需在初始化结构中设置版本字段。
- **跨语言兄弟**：paho-mqtt（Python）、Paho Java（org.eclipse.paho）等，是 [[MQTT]]「跨语言」优势的具体载体。

## 关联连接
- [[MQTT]] — 其实现的协议
- [[同步异步客户端API]] — 同步/异步两套 API 的设计模式
- [[EMQX]] — 官方教程搭配的 Broker
- [[MQTT搭建流程]] — 六步代码对应六阶段骨架
- [[摘要-paho-c客户端教程]] — 来源
