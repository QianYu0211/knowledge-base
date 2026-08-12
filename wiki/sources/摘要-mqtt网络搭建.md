---
title: "摘要-mqtt网络搭建"
type: source
tags: [来源, MQTT, 物联网, 搭建]
sources: [raw/01-articles/20分钟搞定MQTT协议网络搭建及程序实现.txt]
last_updated: 2026-08-12
---

## 核心摘要
- 本资料是「20 分钟搞定 MQTT 协议网络搭建及程序实现」的视频转录，用一个**发布者 + Broker + 订阅者**的小实验，在单台电脑上完整搭建了一套 MQTT 通信网络。
- 核心模型：MQTT 是比 HTTP 更简单的顶层协议，网络呈**中心分布型**——所有客户端（手机/传感器/机器人/手环）作为节点连接中心服务器（Broker），节点间靠 **Topic（主题）+ Payload（负载）** 解耦，发布者无需知道订阅者的 IP，只要主题同名即自动建立连接。
- 搭建过程：以 Ubuntu 虚拟机安装 **EMQX** 作为 Broker（下载一行命令 → 安装 → 浏览器登录 `IP:18083` 后端控制台验证）；Python 侧 `pip` 安装 MQTT 库。
- 发布者五步：导入库 → 创建 client 对象（带唯一 client_id）→ `connect(brokerIP, 1883)` → `loop_start()`（独立线程维护网络通讯）→ `publish(topic, payload)`。连接回调返回 `RC==0` 表示成功，最常见故障是防火墙未放行 1883 端口。
- 订阅者四步：连接部分与发布者完全相同，随后 ① 编写 `on_message` 回调函数 ② 将回调绑定到 client ③ `subscribe(topic)` ④ `loop_start()`。
- `loop` 三种形态：`loop()` 单次监控、`loop_start()` 独立线程循环（最常用，不阻塞主流程）、`loop_forever()` 阻塞式循环。
- 验证手段：Broker 后端控制台实时显示客户端连接数、消息收发频率与断开情况——发布端每秒发送两次、接收端按主题收到消息，即可确认全链路打通。

## 关联连接
- [[MQTT]] — 该资料实验的协议本体
- [[EMQX]] — 该资料使用的 Broker
- [[MQTT搭建流程]] — 该资料演示的搭建过程的抽象
- [[发布订阅模型]] — 主题驱动的解耦模型
