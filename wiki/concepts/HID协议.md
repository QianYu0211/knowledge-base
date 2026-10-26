---
title: "HID协议"
type: concept
tags: [低功耗蓝牙, HID, HOGP, 外设, Report-Map]
sources: [raw/01-articles/【低功耗蓝牙】⑤ HID协议.md]
last_updated: 2026-08-14
---

## 定义

HID（Human Interface Device，人体学接口设备）是描述键盘、鼠标、游戏手柄等输入设备与主机交互方式的协议。早期通过 USB 实现，蓝牙技术出现后以蓝牙为传输层实现无线化；通过低功耗蓝牙（BLE）实现的 HID 称为 **HOGP（HID over GATT Profile）**。BLE HID 通过规定的服务/特性（0x1812 系列 UUID）与 **Report Map** 描述数据格式，实现键鼠、自拍杆、手柄等无线外设。

## 关键信息

- **实现的两个条件**：① 广播数据中广播 HID 服务 UUID、设备外观与设备名称；② GATT 中实现 HID 要求的服务与特性。
- **关键 UUID**：
  - 服务 `0x1812`（HID Service，必须使用）；
  - 外观标识：键盘 `0x03C1`、鼠标 `0x03C2`、游戏手柄 `0x03C3`；
  - `0x2A4A` HID 信息：4 字节（版本 1.1 + 国家码 0 + Flags `0x02`=Normally Connectable）；
  - `0x2A4B` Report Map：描述设备与主机交互数据**每一位的含义**；
  - `0x2A4C` Control Point：必须可写，主机向设备告知自身状态（如电脑休眠）；
  - `0x2A4D` Report：数据交互通道，键盘需输入/输出两个（用 `0x2908` 报告引用区分），鼠标/手柄通常一个即可；
  - `0x2A4E` 协议模式：`0`=Boot 模式、`1`=Report 模式。
- **Boot 模式 vs Report 模式**：
  - **Boot 模式**：BIOS 阶段使用，主机尚未进入系统，交互采用**固定数据格式**（键盘输入 `0x2A22`/输出 `0x2A32`、鼠标输入 `0x2A33`）；
  - **Report 模式**：系统启动后，主机经 `0x2A4E` 通知设备切换，数据格式由 **Report Map** 定义，功能更丰富。
- **键盘 Report（8 字节）**：`Modifier`（1 字节：`0x80`=Ctrl、`0x40`=Shift）+ 保留字节 + 6 个按键槽位（KeyCode：A=0x04、B=0x05、Enter=0x28）。例：`0x80,0x00,0x06,...`=Ctrl+C；`0x40,0x00,0x04,...`=Shift+A。**按下后必须补发抬起数据**，否则系统视为长按。
- **鼠标 Report（4 字节）**：按键位 + X 位移 + Y 位移 + 滚轮（取值约 −127~127）。例：`0x00,0x09,0x09,0x00`=向右上各移 9 像素。
- **自拍杆**：本质是只能调节音量（媒体控制）的简易蓝牙键盘；媒体 Report 单字节，`0x10`=音量−、`0x04`=下一曲。
- **游戏手柄 Report（3 字节）**：X 偏移 + Y 偏移 + 按键（最多 8 键）。
- **工程注意**：配对后再次连接可能需删除设备重新配对；规范的 HID 设备还应包含 DIS（Device Information Service）与 BAS（Battery Service）。

## 关联连接
- [[低功耗蓝牙]] — 所属无线技术
- [[GATT服务]] — HID 服务/特性的承载模型
- [[BLE广播]] — 广播 HID UUID 与外观供主机识别
- [[ESP32]] / [[MicroPython]] — 实现平台
- [[摘要-低功耗蓝牙-hid]] — 来源
