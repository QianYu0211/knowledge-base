---
title: "摘要-freertos打印栈水位"
type: source
tags: [来源, FreeRTOS, 内存, 栈, 调试]
sources: [raw/01-articles/FreeRTOS打印栈水位V2/FreeRTOS打印栈水位V2.md]
last_updated: 2026-08-14
---

## 核心摘要

- 本资料是一篇 FreeRTOS 实验文章，围绕"**打印任务栈水位（High Water Mark）与堆内存状态**"这一调试主题：如何配置宏、移植代码、解读 API 返回值，并深入讲解 MCU 堆栈资源分配。
- **配置三宏**（FreeRTOSConfig.h 必须重新定义）：`configUSE_TRACE_FACILITY 1`、`configUSE_STATS_FORMATTING_FUNCTIONS 1`、`INCLUDE_uxTaskGetStackHighWaterMark 1`，否则相关 API 不可用。
- **四个核心 API**：
  - `vTaskList(pcWriteBuffer)` — 生成所有任务信息表（Name/State/Priority/Stack/Num），返回值以字节为单位；
  - `uxTaskGetStackHighWaterMark(xTask)` — 查询指定任务**历史剩余最小栈内存**（栈水位），返回值以"字"为单位（×4 得字节）；
  - `xPortGetFreeHeapSize()` — 当前剩余堆内存大小；
  - `xPortGetMinimumEverFreeHeapSize()` — 历史剩余最小堆内存（历史最高水位）。
- **堆统计底层**：`xFreeBytesRemaining` 在 `pvPortMalloc()` 分配时减去、`vPortFree()` 释放时加回；`xMinimumEverFreeBytesRemaining` 在小于当前值时更新。二者是静态变量。
- **任务栈开销三要素**（如何确定栈大小）：① 任务局部变量占用；② 最差情况函数嵌套深度；③ 函数参数数量与大小。
- **MCU 堆栈资源**：FreeRTOS 在 MCU 中用**静态数组 `ucHeap[]`** 预先分配堆空间（见 `.map` 文件，位于 SRAM 的 `.bss` 段）；启动文件 `startup.s` 的 `Stack_Size`/`Heap_Size` 与 FreeRTOS 的堆配置是两套独立空间；单个任务最少 128 字（`configMINIMAL_STACK_SIZE`），其中 64 字节用于 TCB、剩余 64 字节用于任务栈。
- **延伸思考**：基于 STM32F411CEU6 验证 `.s` 文件堆栈分配与 FreeRTOS 配置空间的关系；Microlib 的 `malloc()` 与 `pvPortMalloc()` 的区别；`pvPortMalloc()` 后不调 `vPortFree()` 会导致内存泄漏。

## 关联连接
- [[FreeRTOS]] — 该资料讲解的操作系统
- [[栈水位监控]] — 核心主题：High Water Mark 与 API 使用
- [[任务栈大小估算]] — 如何确定任务栈分配大小
- [[FreeRTOS堆内存管理]] — ucHeap 静态数组、pvPortMalloc/vPortFree 与堆统计
- [[任务上下文切换]] — 任务栈是切换时保存寄存器的场所
- [[启动流程]] — 启动文件 Stack/Heap 与 FreeRTOS 堆的关系
- [[STM32]] — 实验平台（STM32F411CEU6）
