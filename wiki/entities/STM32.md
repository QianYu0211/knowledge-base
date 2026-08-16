---
title: "STM32"
type: entity
tags: [MCU, 微控制器, Cortex-M, 嵌入式]
sources: ["raw/01-articles/STM32的启动流程.md", "raw/01-articles/30_STM32_进入低功耗模式/30_STM32_进入低功耗模式.md"]
last_updated: 2026-08-16
---

## 定义

STM32 是意法半导体（ST）基于 **ARM Cortex-M** 内核的 32 位微控制器（MCU）系列，是嵌入式领域最流行的 MCU 平台之一，常作为 [[FreeRTOS]]、[[Bootloader]]、RTOS 应用的宿主。

## 关键信息

- **内核**：Cortex-M0/M3/M4/M7 等，Cortex-M4 引入 FPU 与 DSP 指令；异常模型包括 SysTick、PendSV、SVC 与 NMI/HardFault，天然适配 RTOS。
- **启动模式（BOOT 引脚）**：复位时硬件锁存 BOOT0/BOOT1 电平，决定把哪块物理存储器映射到启动地址 `0x00000000`：
  - `BOOT0=0`：从主 Flash 启动（`0x08000000` 映射，应用正常运行的标准模式）；
  - `BOOT0=1, BOOT1=0`：从系统存储器启动（ST 官方 Bootloader，用于 ISP 串口下载）；
  - `BOOT0=1, BOOT1=1`：从 SRAM 启动（高速调试）。
- **内存映射**：Flash（代码/常量/.data 初值）、SRAM（变量/堆/栈）、部分内核含 TCM 高速紧耦合内存（可放对延迟苛刻的 ISR，见 [[中断优化]]）。
- **上电执行链**：向量表（MSP + Reset_Handler）→ 链接器脚本（LMA/VMA）→ Reset_Handler → C 库 __main（.data 搬运 / .bss 清零）→ main() → OS，详见 [[启动流程]]。
- **开发栈**：HAL/标准外设库、CMSIS（内核访问层）、链接脚本（Keil `.sct` / GCC `.ld`）。
- **OTA 参考工程选型**：**STM32F411CEU6**（Cortex-M4F，外部晶振 25MHz、内部 96MHz）。其内部 Flash 可布局为 Bootloader 区 + App 标志区 `0x8008000`（Sector 2）+ App 运行区 `0x800C000`（Sector 3 起），`MAXAPP_SIZE=464KB`；配合外部 [[W25Q64]] 与 [[Keros]] 加密芯片实现 OTA（见 [[综合-智能手表OTA全流程]]）。
- **低功耗模式**：提供睡眠/停机/待机三种低功耗模式，通过停时钟或断电逐级降低功耗，见 [[低功耗模式]]；进入指令为 Cortex-M 的 [[WFI与WFE]]；供电域（VDD/VDDA/VBAT、1.2V 域、后备域）架构见 [[STM32电源系统]]。

### 案例实现：参考工程中的分区（源码细节）

- 内部 Flash：Bootloader 代码 `0x08000000`（Sector 0–1，32KB）；App 标志区 `0x08008000`（Sector 2，16KB）；App 运行区 `0x0800C000`（Sector 3–7，464KB = `MAXAPP_SIZE`）。
- SRAM：`0x2001FFF0` 起 16B **MYRAM UNINIT 段**（`Bootloader.sct`）承载 `g_JumpInit`，启动跳过清零。

## 关联连接
- [[启动流程]] — 上电到 main 的完整链路
- [[FreeRTOS]] — 最常用 RTOS
- [[Bootloader]] — 引导与 OTA
- [[W25Q64]] / [[Keros]] — OTA 外部硬件
- [[中断优先级]] — NVIC 异常优先级
- [[volatile]] — 硬件寄存器访问规范
- [[低功耗模式]] — 睡眠/停机/待机三种低功耗模式
- [[WFI与WFE]] — 低功耗等待指令入口
- [[STM32电源系统]] — 供电域与 1.2V 域架构
- [[摘要-stm32启动流程]] — 来源
- [[摘要-stm32低功耗模式]] — 来源
