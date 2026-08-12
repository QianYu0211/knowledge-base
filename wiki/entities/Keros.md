---
title: "Keros"
type: entity
tags: [加密芯片, 安全, AES, 密钥管理, I2C]
sources: ["raw/01-articles/OTA面试整理/OTA面试整理.md"]
last_updated: 2026-08-07
---

## 定义

Keros（KEROS）是一款用于安全存储 [[AES]]-256 密钥与初始化向量的**加密芯片型号**（即 [[加密芯片]] 的落地实现），通过 I2C 与主 MCU 通信。密钥与向量**出厂前写入芯片**，[[Bootloader]] 解密固件时才读取——密钥不暴露于固件或普通 Flash。以下细节取自一个智能手表 OTA **参考工程**，该工程选用它承载密钥。

## 关键信息

- **密钥规格**：AES-256，密钥 32 字节 + 向量 16 字节（`KEROS_AES_KEY_LENGTH=32`，`KEROS_AES_VECTOR_LENGTH=16`）。
- **通信接口**：I2C（`keros_i2c_interface.c`），另有 CRC8 校验（`keros_aes_manager.c` 内置 `calculate_crc8`）防止读取数据被篡改/损坏。
- **读取时序**：`InitAESKeyAndIV()` → `keros_aes_init()` 初始化 → `keros_aes_read_key_vector(key, iv)` 读取密钥与向量；也提供缓存读取 `keros_aes_get_key/get_vector` 与认证测试 `keros_aes_authentication_test()`。
- **在 OTA 中的位置**：[[BootManager状态机]] 进入分支前先初始化 AES，随后的 `BootManager_BackToApp()` 用读到的 key/IV 做 AES-256 CBC 解密。
- **标志位选址教训**：早期方案把 OTA 标志位也放加密芯片，因需先配 IIC 再反向初始化、读异常影响升级，后改为放内部 Flash（见 [[加密芯片]]）。

### 案例实现：参考工程中的密钥读取（源码细节）

- EEPROM Page0：`vector(16) + key(32) + CRC8(1) + CRC8_INV(1)`（64B 页内）。
- 读取链路：`keros_aes_init()` → `keros_aes_authentication_test()`（16B 随机挑战-应答，防芯片替换）→ `keros_eeprom_read()` → `verify_aes_data_crc()`（CRC-8，x⁸+x²+x+1）→ key/IV 入 SRAM 缓存。

## 关联连接
- [[加密芯片]] — 所属类别的具体型号
- [[AES]] — 承载的加解密算法
- [[I2C]] — 通信接口
- [[Bootloader]] / [[BootManager状态机]] — 读取方
- [[信息安全目标]] — 保密性硬件基石
- [[摘要-ota面试整理]] — 来源
