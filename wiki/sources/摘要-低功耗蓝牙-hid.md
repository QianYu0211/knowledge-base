---
title: "摘要-低功耗蓝牙-hid"
type: source
tags: [来源, 低功耗蓝牙, HID协议, HOGP, ESP32]
sources: [raw/01-articles/【低功耗蓝牙】⑤ HID协议.md]
last_updated: 2026-08-14
---

## 核心摘要
- 本资料讲解 **蓝牙 HID 协议** 的实现方法，基于 ESP32 实现蓝牙键盘、鼠标、自拍杆、游戏手柄等设备，是学习 BLE HID 的入门参考。
- **HID**（Human Interface Device，人体学接口设备）指键盘/鼠标/手柄等输入设备；早期走 USB，蓝牙技术出现后以蓝牙作传输层实现无线化，通过 BLE 实现的简称 **HOGP（HID over GATT Profile）**。
- **实现条件**：① 广播数据中广播 HID UUID、设备外观、设备名称；② GATT 中实现 HID 要求的服务与特性。
- **关键 UUID**：HID 服务 `0x1812`（必用）；外观标识 `0x03C1` 键盘 / `0x03C2` 鼠标 / `0x03C3` 游戏手柄。
- **HID 特性**：`0x2A4A` HID 信息（4 字节：版本 1.1、国家码 0、Flags 0x02=Normally Connectable）；`0x2A4B` Report Map（描述主机与设备交互数据每一位含义）；`0x2A4C` Control Point（必须可写，主机告知设备状态，如电脑休眠）；`0x2A4D` Report（数据交互，键盘需输入/输出两个，用 `0x2908` 报告引用区分）；`0x2A4E` 协议模式（0=Boot、1=Report）。
- **Boot 模式 vs Report 模式**：BIOS 阶段用 Boot 模式（固定格式：键盘输入 `0x2A22`/输出 `0x2A32`、鼠标输入 `0x2A33`）；进入系统后主机通知设备切到 Report 模式（格式由 Report Map 决定）。
- **键盘 Report（8 字节）**：Modifier（`0x80`=Ctrl、`0x40`=Shift）+ 保留 + 6 键位（KeyCode：A=0x04、B=0x05、Enter=0x28）。例：`0x80,0x00,0x06,...` = Ctrl+C。按下后必须补发抬起数据，否则触发长按。
- **鼠标 Report（4 字节）**：按键位 + X 位移 + Y 位移 + 滚轮（各 −127~127/128）。**自拍杆**本质是只能调音量（媒体控制）的简易键盘；媒体 Report 单字节，`0x10`=音量−、`0x04`=下一曲。
- **游戏手柄 Report（3 字节）**：X 偏移 + Y 偏移 + 按键（最多 8 键）。
- 已知注意点：配对成功后需删除设备重新配对才能再次连接；正常 HID 设备还应包含 DIS（Device Information Service）与 BAS（Battery Service）。

## 关联连接
- [[HID协议]] — HID/HOGP 协议模型
- [[GATT服务]] — HID 服务与特性的承载
- [[低功耗蓝牙]] — 主题实体
- [[ESP32]] / [[MicroPython]] — 实践平台
- [[BLE广播]] — 广播 HID UUID 与外观供识别
