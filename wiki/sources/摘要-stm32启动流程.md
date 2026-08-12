---
title: "摘要-stm32启动流程"
type: source
tags: [来源, 原始文件, STM32, Cortex-M, 启动流程]
sources: ["raw/01-articles/STM32的启动流程.md"]
last_updated: 2026-08-03
---

## 核心摘要

STM32（Cortex-M）启动流程七阶段：**阶段0 硬件自举**——芯片复位锁存 BOOT0/BOOT1 引脚，决定把主 Flash（0x08000000）、系统存储器还是 SRAM 映射到启动地址 0x00000000。**阶段1 内核复位与中断向量表**——CPU 从向量表首条取主堆栈指针 MSP，第二条取 Reset_Handler 地址跳转执行。**阶段2 链接器脚本**——定义 .data 段的 LMA（Flash 仓库）与 VMA（RAM 新家）、.bss 段 RAM 空地、堆栈位置。**阶段3 Reset_Handler**——调用 SystemInit() 配时钟后跳转 C 库入口 __main。**阶段4 __main 搬家公司**——按蓝图 memcpy 搬运 .data、memset 清零 .bss，再调用 main()。**阶段5 main()**——外设初始化、创建 OS 对象、调用 vTaskStartScheduler()（不返回）。**阶段6 OS 接管**——配 SysTick 心跳、触发 SVC 异常进入处理模式、vPortSVCHandler 加载首任务上下文并把栈指针从 MSP 切到 PSP、返回线程模式运行首任务；此后 SysTick→PendSV 驱动后续任务切换。

## 关联连接
- [[STM32]] — 芯片主体
- [[启动流程]] — 七阶段机制详解
- [[Bootloader]] — VTOR 重定位与自定义引导
- [[FreeRTOS]] — 阶段6 的 OS 接管
- [[任务上下文切换]] — SysTick/PendSV 驱动的切换
- [[摘要-freertos任务上下文切换]] — 切换机制续
