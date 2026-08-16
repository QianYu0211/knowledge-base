---
title: "MicroPython"
type: entity
tags: [编程语言, 嵌入式, Python, 开发环境]
sources: [raw/01-articles/【低功耗蓝牙】广播数据分析.md, raw/01-articles/【低功耗蓝牙】② 状态切换和事件处理.md, raw/01-articles/【低功耗蓝牙】③ 蓝牙服务和特性的创建.md, raw/01-articles/【低功耗蓝牙】④ 蓝牙MIDI协议.md, raw/01-articles/【低功耗蓝牙】⑤ HID协议.md]
last_updated: 2026-08-14
---

## 定义

MicroPython 是面向微控制器的 Python 3 精简实现，让开发者用脚本语言快速驱动底层硬件。其在低功耗蓝牙开发中提供 `ubluetooth` 库，以极简 API 完成 BLE 广播、服务注册与数据交互，大幅降低 BLE 外设的开发门槛。

## 关键信息

- **`ubluetooth` 模块**：ESP32 上操作 BLE 的标准库，提供 `BLE()` 设备对象、`UUID()`、`gap_advertise()`、`gatts_register_services()`、`gatts_write/read/notify()`、`irq()` 等接口。
- **事件回调**：`ble.irq(ble_irq)` 注册中断处理函数，按事件 ID 分发——`1`=已连接、`2`=断开连接（须重新广播以恢复可发现）、`3`=收到新写入消息。
- **广播 API**：`ble.gap_advertise(间隔ms, adv_data=..., resp_data=...)` 同时支持广播数据与扫描响应数据。
- **服务注册**：通过 UUID + 权限标志（`FLAG_READ/FLAG_WRITE/FLAG_NOTIFY`）定义特性，组合为服务集合后一次性注册到 GATT 协议栈。
- **典型用例**：BLE 广播、状态机演示、服务/特性数据交互、BLE MIDI、HID 外设（键盘/鼠标/自拍杆/游戏手柄）。

## 关联连接
- [[ESP32]] — 主要运行的芯片平台
- [[低功耗蓝牙]] — 被操作的无线技术
- [[GATT服务]] — 服务/特性注册与数据交互 API 的底层模型
- [[BLE广播]] / [[BLE状态机]] — API 对应的协议机制
- [[摘要-低功耗蓝牙-广播数据分析]] / [[摘要-低功耗蓝牙-状态切换]] / [[摘要-低功耗蓝牙-服务与特性]] / [[摘要-低功耗蓝牙-midi]] / [[摘要-低功耗蓝牙-hid]] — 来源
