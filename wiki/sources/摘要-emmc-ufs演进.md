---
title: "摘要-emmc-ufs演进"
type: source
tags: [来源, 原始文件, eMMC, UFS, SSD, 存储]
sources: ["raw/01-articles/几种闪存技术：eMMC、UFS2.1、UFS3.0、SSD_ufs vs.emmcperformance scaling-CSDN博客.md"]
last_updated: 2026-08-19
---

## 核心摘要

CSDN 技术文章，梳理存储设备**按使用场景分化**的路线：SSD 面向 PC/服务器，eMMC/UFS 面向移动设备；再讲 MMC → eMMC → UFS 的演进史与接口差异。核心洞察：**eMMC 把"高密度 NAND + 控制器"封装成一颗 BGA 芯片，把坏块管理/ECC/磨损均衡等 Flash 管理技术内化，用户无需关心晶圆制程；UFS 用高速串行差分接口 + 全双工取代 eMMC 的并行半双工，成为移动存储新趋势**。文中还提到 EEPROM 的历史地位——1988 年 Intel NOR flash 的出现"彻底改变了原先由 EPROM 和 EEPROM 一统天下的局面"。

## 关键信息

### 场景分化：SSD vs eMMC/UFS
- **SSD**：取代 PC/服务器 HDD，需超大容量（百 GB~TB）、极高并行性、兼容 SATA/PCI，对功耗体积不敏感。
- **eMMC/UFS**：针对移动设备，适当容量与性能、对功耗体积极其敏感、遵循一定接口标准即可。

### 闪存演进时间线
- 1988 年 Intel 首发 NOR flash → 改变 EPROM/EEPROM 一统天下的局面。
- 1989 年东芝发表 NAND flash → 强调低成本、高容量、磁盘式接口升级。
- 制程与存储单元发展（SLC → MLC → TLC）让容量暴涨、成本骤降 → eMMC 应运而生。
- NAND 行业摩尔定律周期约 12 个月（比通用摩尔定律 18 个月更快）。

### SLC / MLC / TLC
| 类型 | 每单元位数 | 特点 | 擦写寿命 |
|---|---|---|---|
| SLC | 1 bit/cell | 读写快、寿命长、价格约 MLC 三倍以上 | 约 10 万次 |
| MLC | 2 bit/cell | 速度/寿命/价格一般 | 约 3000–10000 次 |
| TLC | 3 bit/cell | 速度慢、寿命短、最便宜 | 约 500 次 |

### eMMC：内化 Flash 管理的封装
- MMC 是闪存卡标准，定义架构与访问接口/协议；eMMC（embedded MultiMediaCard）是 MMC 的嵌入式拓展。
- 结构：**高密度 NAND Flash + MMC Controller 封装进一颗 BGA 芯片**，采用统一 MMC 标准接口。
- 内部已含 Flash 管理技术：**错误探测和纠正（ECC）、平均擦写（磨损均衡）、坏块管理、掉电保护**。
- 用户无需担心内部晶圆制程/工艺变化；单颗芯片为主板节省空间。

### eMMC → UFS 的驱动力
- eMMC 协议多年未更新、并行 8 总线数据时序到瓶颈；并行-串行转换使速率难以提升。
- UFS（Universal Flash Storage）：高性能接口，高速串行 + 全双工，面向智能手机/平板/汽车等低功耗应用。

### eMMC vs UFS 接口对比
| 维度 | eMMC | UFS |
|---|---|---|
| 传输形式 | 半双工（同一时刻只能读或写） | 全双工（可同时读写） |
| 物理层 | 单线传输（需考虑噪声容限，信号电平高） | 差分传输（抗干扰、低电平高速） |
| 总线 | 8 条数据总线复用 + 命令线 + STROBE 线（eMMC5.1 HS400 新增，自适应时序防温度漂移） | 两组差分线 + 一条时钟线（类似 USB3.0），IO 少、配置简单 |
| 同步方式 | 同步（需时钟参考） | 同步（需时钟参考） |

### UFS3.0 亮点（2018 JEDEC 发布 HS-G4）
- 单通道带宽 11.6Gbps，双通道 23.2Gbps ≈ 2.9GB/s（UFS2.1 为 1450MB/s，翻倍）。
- 供电电压降低：VCC 2.9V → 2.5V，VCCQ 1.8V → 1.2V，直接降低功耗与发热。
- **Write Turbo**：SLC 动态加速——写入先落更快的 SLC，空闲时主控再把数据搬入较慢的 TLC（类比 CPU 缓存与内存）。

## 关联连接
- [[Emmc]] — 本素材编译出的概念页
- [[Ufs]] — 本素材编译出的概念页
- [[NorFlash]] / [[NandFlash]] — 闪存原理与对比（NOR/NAND 本体知识）
- [[摘要-闪存原理与nor-nand对比]] — 配套素材摘要
- [[W25Q64]] — 嵌入式侧常用的外置 NOR Flash 对照
- [[综合-rtos与linux]] — Linux 侧块设备/MTD 的存储管理背景
