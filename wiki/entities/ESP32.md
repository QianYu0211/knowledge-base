---
title: "ESP32"
type: entity
tags: [芯片, 物联网, Wi-Fi, 蓝牙, 乐鑫]
sources: [raw/01-articles/【低功耗蓝牙】广播数据分析.md, raw/01-articles/【低功耗蓝牙】② 状态切换和事件处理.md, raw/01-articles/【低功耗蓝牙】③ 蓝牙服务和特性的创建.md, raw/01-articles/【低功耗蓝牙】④ 蓝牙MIDI协议.md, raw/01-articles/【低功耗蓝牙】⑤ HID协议.md]
last_updated: 2026-08-14
---

## 定义

ESP32 是乐鑫（Espressif）推出的低成本、低功耗 Wi-Fi + 蓝牙 SoC 系列，内置 **BLE（4.2/5.0）** 与经典蓝牙，广泛用于物联网节点、智能硬件与电子外设。在低功耗蓝牙系列资料中，ESP32 作为实践平台运行 MicroPython 固件，演示广播、GATT 服务、BLE MIDI 与 HID 设备的完整实现。

## 关键信息

- **双无线能力**：2.4GHz Wi-Fi + 蓝牙/BLE 双模，单芯片即可实现联网与近距离数据交互。
- **MicroPython BLE 编程**：通过 `ubluetooth` 模块操作 BLE——`ble = ubluetooth.BLE()` 创建设备、`ble.active(True)` 打开、`ble.gap_advertise()` 设置广播并开始广播、`ble.irq()` 注册事件回调、`ble.gatts_register_services()` 注册服务、`ble.gatts_write()/gatts_read()/gatts_notify()` 读写与推送特性数据。
- **开发环境**：资料使用 uPyCraft 开发环境 + MicroPython 固件（HID 示例要求 V1.16 以上），调试搭配 nRF Connect 手机 APP 或「谷雨蓝牙」微信小程序。
- **典型应用**：广播数据验证、服务/特性创建、电池/温湿度传感器、BLE MIDI 电子乐器、蓝牙键盘/鼠标/自拍杆/游戏手柄（HID）。

## 关联连接
- [[低功耗蓝牙]] — 实现的无线通信技术
- [[MicroPython]] — 主要编程平台
- [[nRFConnect]] — 调试与验证工具
- [[BLE广播]] / [[BLE状态机]] / [[GATT服务]] — 涉及的核心机制
- [[HID协议]] / [[蓝牙MIDI协议]] — 典型应用 Profile
- [[摘要-低功耗蓝牙-广播数据分析]] / [[摘要-低功耗蓝牙-状态切换]] / [[摘要-低功耗蓝牙-服务与特性]] / [[摘要-低功耗蓝牙-midi]] / [[摘要-低功耗蓝牙-hid]] — 来源
