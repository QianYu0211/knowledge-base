---
title: "摘要-stm32低功耗模式"
type: source
tags: [来源, 原始文件, STM32, 低功耗, HAL]
sources: [raw/01-articles/30_STM32_进入低功耗模式/30_STM32_进入低功耗模式.md]
last_updated: 2026-08-16
---

## 核心摘要

- 本文是 STM32 低功耗模式的实践教程，覆盖**睡眠（Sleep）、停机（Stop）、待机（Standby）**三种模式的原理与 HAL 实现。
- 首先从芯片供电管脚（VCC/VDD/VSS/VBAT）讲起，梳理 STM32 电源管理系统三大区域：**VDDA 模拟供电域、VDD+1.2V 数字供电域、VBAT 后备供电域**，并解释调压器与 1.2V 域在不同模式下的工作状态。
- 三种模式逐级降低功耗：Sleep 仅停 CPU 时钟；Stop 关闭 1.2V 域所有时钟但保留内核寄存器与 SRAM；Standby 切断整个 1.2V 域电源，仅后备域（RTC/备份寄存器）维持。
- 讲解 Cortex-M 的 **WFI（Wait For Interrupt）与 WFE（Wait For Event）** 两条指令作为进入低功耗的入口及其唤醒条件差异。
- 提供 HAL 实现要点：`HAL_PWR_EnterSLEEPMode` / `HAL_PWR_EnterSTOPMode` / `HAL_PWR_EnterSTANDBYMode`，以及 Stop 唤醒后需重配系统时钟、Standby 需唤醒引脚（PA0）上升沿触发等工程细节。

## 关联连接
- [[低功耗模式]] — 三种低功耗模式的核心概念
- [[WFI与WFE]] — 进入低功耗的两条 Cortex-M 指令
- [[STM32电源系统]] — 供电域与 1.2V 域基础
- [[STM32]] — 平台实体
- [[启动流程]] — Cortex-M 运行与复位上下文
