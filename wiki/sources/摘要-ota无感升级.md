---
title: "摘要-ota无感升级"
type: source
tags: [来源, 原始文件, OTA, Bootloader, 固件升级]
sources: ["raw/01-articles/OTA无感升级流程.md"]
last_updated: 2026-08-03
---

## 核心摘要

OTA 无感升级的核心是一个存储在 Flash 中的 **"App 标识"** 接力棒，在 App 与 Bootloader 之间传递控制权。**App 阶段**后台静默准备：监控线程按设备繁忙度决策低速/高速下载；OTA 线程用**非阻塞 Ymodem** 下载，接收采用 `中断 + DMA + AB 双缓冲`（DMA 存满 A 后中断切到 B，CPU 经**消息队列**把 A 交给 Flash 线程写入外部 Flash 下载区 A 区）；Ymodem 断点续传保证断网/断电后从上次位置继续。准备完成后 App 把标识改为"待更新"并软复位。**Bootloader 阶段**经历多次自动重启完成安全安装：①验证下载区固件的**解密 + CRC 校验**，失败则放弃更新避免变砖；②A 区搬运到备份区 B 区；③擦除 App 运行区、从 B 区逐块解密校验写入、最后全量 CRC 校验；④标识置为"有效 App"后启动看门狗跳转运行新固件。附 OTA 状态机（IDLE→CHECK_UPDATE→DOWNLOADING→VERIFYING→UPDATING→UPDATED/ROLLBACK）与回滚/防回滚机制（失败标记 BAD 切备份区；同版本连续失败 3 次禁用）。

## 关联连接
- [[OTA升级]] — 无感升级机制
- [[Bootloader]] — 安装执行者
- [[YMODEM]] — 下载传输协议
- [[DMA双缓冲]] — 下载接收黄金组合
- [[消息队列]] — 线程间交接数据
- [[AES]] / [[CRC]] — 固件解密与完整性校验
- [[信息安全目标]] — 升级安全性的四目标支撑
