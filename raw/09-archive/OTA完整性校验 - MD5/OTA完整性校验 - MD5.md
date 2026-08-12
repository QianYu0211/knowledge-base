# OTA完整性校验 \- MD5

[MD5\.zip](图片和附件/MD5.zip)

# 学习目标

理解MD5是不可逆哈希（非加密），掌握其校验用途

掌握文件/文本/标准输入的MD5计算方法与工具使用

掌握将MD5追加到文件尾（方案A/方案B）与旁文件（方案C）的生成与校验

了解在固件分发、设备升级、文件下载等场景的应用方式与注意事项



# 知识学习

1. 什么是MD5

输入任意长度数据，输出固定128位（16字节）摘要；常以32位十六进制字符串表示

主要用途：完整性校验（是否被意外篡改或传输损坏）

局限：存在碰撞攻击，不适用于安全防伪/签名；涉及安全请选择SHA‑256或数字签名



2. MD5与“加密”的区别

MD5不可逆，只能“重新计算并比对”；不存在“解密MD5”的概念

加密/解密（如AES/RSA）用于保密或身份认证，语义完全不同



3. 计算MD5的常见方式

文件MD5：从头到尾读取文件并流式更新哈希状态，输出摘要

文本MD5：将文本按UTF‑8编码为字节后计算摘要

标准输入MD5：从stdin读取数据后计算摘要（可选择按二进制处理）



4. 追加与存放位置（方案A/B/C）

方案A（原始16字节）：把16字节MD5原文直接追加到文件末尾；位置是最后16字节区间\[N‑16, N\)

方案B（ASCII32字节）：把32个ASCII十六进制字符追加到文件末尾；便于人工查看

方案C（旁文件）：不改动原文件，单独生成\<文件\>\.md5旁文件保存摘要



# 实践/实验内容

1. 环境准备（Windows PowerShell）

安装Python 3；在目录 e:\\01\_LX\\06\_ec\_s100\_watch\\ec\_s100\_watch1\\07\_Tools\\MD5 下使用提供的 md5\.py



2. 基础命令

计算文件MD5：`py md5.py hash .\full.bin`（仅输出摘要加`--raw`）

计算文本MD5：`py md5.py hash -t "Hello"`（仅输出摘要加`--raw`）

从标准输入计算：`py md5.py hash --stdin`（二进制加`--binary-stdin`）



3. 生成带尾部MD5的文件

方案A（原始16字节，推荐机器消费）：

生成新文件：`py md5.py append .\full.bin --format raw`（输出默认为`full.bin.md5.bin`）

直接写回原文件：`py md5.py append .\full.bin -o .\full.bin --format raw`

方案B（ASCII32字节）：`py md5.py append .\full.bin --format hex`

方案C（旁文件）：`py md5.py append .\full.bin --format sidecar`（生成`full.bin.md5`）



4. 校验尾部MD5

自动识别raw/hex：`py md5.py verify .\full.bin`

强制模式：`py md5.py verify .\full.bin --format raw` 或 `--format hex`

C版本校验器（需先编译）：

MSVC：`cl /O2 md5_verify.c`

MinGW：`gcc -O2 -o md5_verify.exe md5_verify.c`

运行：`md5_verify.exe .\full.bin [--format auto|raw|hex]`



5. 打印尾部16字节（方案A的原始MD5）为十六进制

PowerShell示例：

`$p = ".\full.bin"`

`$fs = [System.IO.File]::Open($p, [System.IO.FileMode]::Open)`

`$fs.Seek(-16, [System.IO.SeekOrigin]::End) | Out-Null`

`$buf = New-Object byte[] 16`

`$fs.Read($buf,0,16) | Out-Null; $fs.Close()`

`($buf | ForEach-Object { $_.ToString("X2") }) -join ""`



# 应用场景

1. 固件分发与设备升级

发布端按方案A将MD5原始16字节追加到固件尾部；设备端读取最后16字节作为期望值，对前N‑16字节重算MD5比对

优点：无需额外协议即可实现一致性校验；缺点：设备需按尾部结构解析



2. 文件下载完整性校验

分发文件并附带MD5（方案B或方案C）；接收端下载后本地重算MD5，与附带摘要比对，确认未损坏



3. 自动化流水线

构建阶段自动生成带MD5尾部的产物（方案A），或生成旁文件（方案C）；部署前或设备烧录前进行校验，保证产物正确



4. 协议化尾部结构（扩展）

在严格设备协议中，可设计尾部为 `{魔数 | 版本 | 长度 | MD5}`，便于解析与升级兼容；本仓库工具可生成MD5，协议解析由设备端实现



# 面试问题准备

1. MD5与加密的区别与适用场景

2. 为什么MD5不适用于安全防伪？如何选择SHA‑256/签名

3. 方案A与方案B的取舍：容量、可读性、设备解析难度

4. 如何在设备端实现高效校验与异常处理（读取尾部、分块计算、超时与错误码）

5. 追加MD5后对文件大小与偏移的影响（尾部区间与N‑16/N‑32的计算）

    

# Q\&A

1. 能“解密”MD5吗？

不能。MD5是不可逆哈希；校验流程是“重新计算并比对”而非解密



2. 追加MD5命令提示“‘\<’运算符是为将来使用而保留的”？

PowerShell中`<`/`>`是重定向符号；请使用真实文件名，不要写尖括号占位符。例如：`py md5.py append .\full.bin --format raw`



3. 如何只打印摘要字符串而不带说明文字？

在`hash`子命令后添加`--raw`：`py md5.py hash .\full.bin --raw`



4. 路径包含空格如何处理？

用引号包裹：`py md5.py append "E:\path with space\full.bin" --format raw`



5. 设备严格读取固定长度文件怎么办？

使用方案C（旁文件）；或按设备协议设计尾部结构，由设备按协议解析



参考与仓库位置

工具脚本：`md5.py`（追加与校验实现）

校验器：`md5_verify.c`（读取尾部16/32字节并比对）

方案说明：`MD5_教学文档.md` 中的A/B/C流程

