---
title: "摘要-低功耗蓝牙-midi"
type: source
tags: [来源, 低功耗蓝牙, 蓝牙MIDI协议, MIDI, ESP32]
sources: [raw/01-articles/【低功耗蓝牙】④ 蓝牙MIDI协议.md]
last_updated: 2026-08-14
---

## 核心摘要
- 本资料介绍 **蓝牙 MIDI 协议**：乐器与手机/电脑间无线数据传输的协议格式，以及基于 ESP32 的电子乐器实现。
- **MIDI**（Musical Instrument Digital Interface，乐器数字接口）：20 世纪 80 年代初为解决电声乐器间通信提出的协议，传输的是乐器动作（按键按下/抬起、力度等）而非音频本身。
- **蓝牙 MIDI 历史**：传统 MIDI 用线缆实现、不便移动；Apple 2014 年设计支持 BLE 的 MIDI 协议，MMA（MIDI 制造商协会）2015 年 6 月将其列为行业标准。
- **两个必要条件**：① 广播数据中广播 128-bit 服务 UUID `03B80E5A-EDE8-4B33-A751-6CE34EC4C700`；② GATT 中包含该 UUID 的服务，内含特性 `7772E5DB-3868-4112-A1A9-F2669D106BF3`（权限 Read/Write/Notify）。不广播 MIDI 服务 UUID 则不会被识别为 MIDI 设备。
- **MIDI 指令格式（5 字节）**：前 2 字节时间戳（通常 `0x80 0x80`）→ 第 3 字节设备动作（命令）→ 第 4/5 字节参数。例：`80 80 90 48 63` = 以力度 99（0x63）按下 C4（0x48）；`80 80 80 48 00` = 抬起 C4。
- **音符编码**：每个音符有对应 MIDI 编码（如 C4=0x48），通过查表映射。
- 实现：ESP32 MicroPython 读取按键，`ble.gatts_notify()` 发送按下/抬起指令；配合手机 APP（iOS 库乐队/Echo Piano Lite、Android 泡泡钢琴）即可演奏。

## 关联连接
- [[蓝牙MIDI协议]] — BLE 承载 MIDI 的协议模型
- [[MIDI]] — 传统有线 MIDI 协议本体
- [[GATT服务]] — 作为 BLE 传输层
- [[低功耗蓝牙]] — 主题实体
- [[ESP32]] / [[MicroPython]] — 实践平台
