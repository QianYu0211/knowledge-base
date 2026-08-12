---
title: "摘要-freertos任务上下文切换"
type: source
tags: [来源, 原始文件, FreeRTOS, RTOS, 任务调度]
sources: ["raw/01-articles/FreeRTOS任务上下文切换.md"]
last_updated: 2026-08-03
---

## 核心摘要

FreeRTOS 的任务上下文切换是一场职责分离的"舞台剧"：**计时员**（SysTick 中断 `xPortSysTickHandler`）只负责报时；**选角导演**（`xTaskIncrementTick` + `vTaskSwitchContext`）只负责决策；**舞台监督**（`xPortPendSVHandler`）在幕后精准保存/恢复演员的寄存器道具。核心数据结构是 **TCB**（`pxTopOfStack` 道具清单指针 + `xStateListItem` 状态牌，状态牌嵌入 TCB 实现零开销换列表）与**链表**（就绪区/延迟区，`pxIndex` 记录导演"手指"上次停留位置实现轮转）。切换流程七步：Tick 钟声 → 导演唤醒延迟任务入就绪区 → 举换角旗（新任务优先级不低于当前）→ PendSV 保存旧上下文 → 导演更新 `pxCurrentTCB`（`listGET_OWNER_OF_NEXT_ENTRY` 移动手指 + 经 `pvOwner` 认主）→ 恢复新上下文 → 新任务登台。触发时机有三：时间片耗尽、高优先级任务就绪、主动 `taskYIELD()`。

## 关联连接
- [[FreeRTOS]] — 所属操作系统
- [[任务上下文切换]] — 机制详解
- [[任务调度]] — 选角决策策略
- [[消息队列]] — RTOS 另一核心同步机制（对比）
- [[信号量]] — RTOS 同步机制（对比）
- [[摘要-串口环形缓冲]] — 同一生态的接收机制
