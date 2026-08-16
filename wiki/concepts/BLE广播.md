---
title: "BLE广播"
type: concept
tags: [低功耗蓝牙, 广播, AD Structure, AD Type]
sources: [raw/01-articles/【低功耗蓝牙】广播数据分析.md]
last_updated: 2026-08-14
---

## 定义

BLE 广播是从机设备在 3 个广播信道上按固定数据格式周期性发射数据包的行为，是低功耗蓝牙**无连接通信**的基石——让主机无需连接即可发现设备、了解其名称、外观与能力（如支持的 Profile）。广播数据容量有限，超出的次要信息可放入**扫描响应**。

## 关键信息

### 广播 PDU 类型（链路层）

广播 PDU 类型由链路层广播包的 PDU Type 字段标识，决定该广播"能否被扫描 / 能否被连接"：

| PDU 类型 | 完整名称 | 可扫描 | 可连接 | 典型用途 |
|---------|------|:---:|:---:|------|
| `ADV_IND` | Connectable Undirected Advertising（可连接无向广播） | ✅ | ✅ | 通用广播，最常见 |
| `ADV_DIRECT_IND` | Connectable Directed Advertising（可连接定向广播） | ❌ | ✅ | 只发指定目标设备，快速重连 |
| `ADV_NONCONN_IND` | Non-Connectable Undirected Advertising（不可连接无向广播） | ❌ | ❌ | 纯信息广播（如信标 Beacon），不建立连接 |
| `ADV_SCAN_IND` | Scannable Undirected Advertising（可扫描无向广播） | ✅ | ❌ | 仅可被扫描获取更多信息 |

- 广播信道上还传输三种相关 PDU：`SCAN_REQ`（主机扫描请求）、`SCAN_RSP`（从机扫描响应）、`CONNECT_REQ`（主机连接请求）；连接成功后转入数据信道通信。
- MicroPython `ble.gap_advertise()` 默认即 `ADV_IND`（可连接无向广播）；"广播 vs 扫描响应" 正是 `SCAN_REQ`/`SCAN_RSP` 的行为。
- 蓝牙 5.0 引入**扩展广播**（`ADV_EXT_IND` + 辅助数据信道 + 周期广播），支持更大数据负载。
- ⚠️ 区分两个"类型"：**广播 PDU 类型**（本小节，决定能否连/能否扫）与 **AD Type**（下文数据内容类型，描述这段数据是什么）是两层不同概念。

### 广播数据内容

- **广播包容量**：广播包最大 **37 字节**，其中设备地址占 6 字节，可用数据仅 **31 字节**。
- **AD Structure（广播数据单元）**：31 字节可用区被切分为若干 AD Structure，每段由三部分组成：
  - `Length`（1 字节）：`AD Type + AD Data` 的总长度；
  - `AD Type`（1 字节）：数据类型标识，由蓝牙技术联盟（SIG）分配；
  - `AD Data`（n 字节）：数据内容。
  - 解析时依 Length 顺序切分，即可还原各数据段。
- **常见 AD Type**：
  - `0x01` 设备标识（Flags）——低两位比特组合表示发现模式/是否仅 LE，`0x06`（Bit1+Bit2）= 普通发现模式且仅支持 LE；
  - `0x02/0x03` 服务 UUID 部分/完整（16-bit）；
  - `0x08/0x09` 短/完整设备名称；
  - `0x0A` 发射功率（Tx Power）；
  - `0xFF` 厂商自定义数据。
- **设备名称编码**：采用 **UTF-8**，一个汉字占 3 字节，因此设备名可设置为任意语言文字。
- **广播 vs 扫描响应**：
  - 广播：从机**主动**周期性发出，用于基本发现；
  - 扫描响应：主机收到广播后发送**扫描请求**，从机回复**扫描响应**；两者数据格式完全一致，扫描响应用于分担 31 字节装不下的次要数据。
  - MicroPython 中用 `ble.gap_advertise(间隔, adv_data=..., resp_data=...)` 分别设置。
- **服务 UUID 广播的意义**：在广播中携带服务 UUID（如 HID `0x1812`、BLE MIDI `03B80E5A-...`）可让主机识别设备能力并决定是否连接，是 HID/蓝牙 MIDI 等 Profile 的必要前提。

## 关联连接
- [[低功耗蓝牙]] — 所属无线技术
- [[BLE状态机]] — 广播态是状态机中的一种状态
- [[GATT服务]] — 广播中携带的服务 UUID 与连接后数据交互的关联
- [[HID协议]] / [[蓝牙MIDI协议]] — 依赖广播 UUID 识别设备类型
- [[ESP32]] / [[MicroPython]] / [[nRFConnect]] — 实现与调试
- [[摘要-低功耗蓝牙-广播数据分析]] — 来源
