---
title: "FreeRTOS堆内存管理"
type: concept
tags: [FreeRTOS, 堆, 内存, 内存管理, 静态数组]
sources: [raw/01-articles/FreeRTOS打印栈水位V2/FreeRTOS打印栈水位V2.md]
last_updated: 2026-08-14
---

## 定义

FreeRTOS 在 MCU 中**不依赖 C 库的 malloc**，而是用**静态数组 `ucHeap[]`** 预先划出一块"FreeRTOS 堆"（大小由 `configTOTAL_HEAP_SIZE` 决定），内核用 `pvPortMalloc()`/`vPortFree()` 在此堆上分配任务栈、TCB、队列、信号量等对象。该数组放在 SRAM 的 `.bss` 段（见 `.map` 文件），与启动文件的堆（`Heap_Size`）是**两套独立空间**。

## 关键信息

### 堆内存模型

- **静态大数组**：`ucHeap[configTOTAL_HEAP_SIZE]` 编译期就分配在 RAM 的 `.bss` 段，不依赖 C 运行库初始化；
- **分配入口**：内核对象（任务、队列等）创建时统一走 `pvPortMalloc()`，释放走 `vPortFree()`；
- **与 Microlib `malloc()` 的区别**：
  - C 库 `malloc()` 使用启动文件 `Heap_Size`（如 `0x800`）划出的 HEAP 区，管理方式随库实现；
  - `pvPortMalloc()` 使用 `configTOTAL_HEAP_SIZE` 对应的 `ucHeap`，是 FreeRTOS 自己实现的内存堆，**不经过 C 库**。
- **内存泄漏**：`pvPortMalloc()` 分配后不调 `vPortFree()`，堆空间只减不增，长时间运行后堆耗尽、后续创建任务/队列失败（`pdPASS` 之外的返回值）。

### 堆水位统计（三个关键量）

| 量 | 语义 |
|---|---|
| `configTOTAL_HEAP_SIZE` | FreeRTOS 堆总大小（字节） |
| `xPortGetFreeHeapSize()` | **当前**剩余堆内存（字节），返回 `xFreeBytesRemaining` |
| `xPortGetMinimumEverFreeHeapSize()` | **历史最低**剩余堆内存（字节），返回 `xMinimumEverFreeBytesRemaining` |

### 底层实现（`xFreeBytesRemaining` 与 `xMinimumEverFreeBytesRemaining`）

- 两个**静态变量**，在内存管理源文件中维护：
  - `xFreeBytesRemaining`：每次 `pvPortMalloc()` 分配时**减去**该块大小；每次 `vPortFree()` 释放时**加回**（从空闲链表中查找删除块并归还）；
  - `xMinimumEverFreeBytesRemaining`：当**小于当前 `xFreeBytesRemaining`** 时更新（记录历史最低水位）。
- 注意：只统计"剩余字节数"，**不反映碎片化（fragmentation）**——源码注释明确说明 "says nothing about fragmentation"。

### 典型排查流程

1. `xPortGetMinimumEverFreeHeapSize()` 看历史最低剩余是否接近 0 → 堆耗尽；
2. 比对 `xPortGetFreeHeapSize()` 当前值与历史最低值 → 判断是否有突发大分配；
3. 内存持续减少但无增长 → 定位未释放的 `pvPortMalloc()`（内存泄漏）；
4. 若碎片化严重（总量够但分配失败）→ 需要更大堆或改用不同 heap 方案。

## 关联连接
- [[FreeRTOS]] — 内核的内存来源
- [[栈水位监控]] — 堆水位与栈水位的对照排查
- [[任务栈大小估算]] — 任务栈也来自该堆
- [[启动流程]] — 启动文件 Heap/Stack 与 FreeRTOS 堆的关系
- [[STM32]] — 实验平台（SRAM `.bss` 段）
- [[摘要-freertos打印栈水位]] — 来源
