---
title: "CAN通信基础：总线关闭（BUS OFF）的概念和处理方式"
source: "https://blog.csdn.net/wendy_mooncat/article/details/144477239"
author:
  - "[[wendy_mooncat]]"
published: 2024-12-16
created: 2026-08-14
description: "文章浏览阅读1.4w次，点赞37次，收藏142次。CAN总线BUS OFF状态的概念介绍和恢复策略_can busoff"
tags:
  - "clippings"
---
## 1 BUS OFF的概念

在CAN通信中，BUS OFF是指节点由于发送错误计数器超出一定数量，而被总线“关闭”的状态。当节点进入BUS OFF状态时，它将无法再发送或接收任何消息，直到被重置或自动恢复。BUS OFF是 CAN 通信提供的一种错误处理机制，它可以防止错误节点继续影响整个网络。

## 2 BUS OFF故障

### 2.1 通信节点状态

CAN节点的状态转移包括以下几个阶段：  
![节点状态转移图](https://i-blog.csdnimg.cn/direct/a0e9cf7c9e294fb48e1c701ec89b3f09.png#pic_center)

- **主动错误（Error Active）** ：
	- ECU上电或重启后，默认进入主动错误状态，TEC（发送错误计数器）和REC（接收错误计数器）清零；
		- 主动错误状态下，通信节点可以正常发送和接收消息；
		- 主动错误状态下，通信节点若检测到错误，则发送 **主动错误标志（连续6个显性位）** 。
- **被动错误（Error Passive）** ：
	- 当ECU的TEC或REC计数大于127，ECU自主动错误状态进入被动错误状态；
		- 被动错误状态下，通信节点可以发送和接收消息，但会受到一定的限制（如传输暂停）；
		- 被动错误状态下，通信节点若检测到错误，则发送 **被动错误标志（连续6个隐性位）** 。
		- 被动错误状态下，若TEC和REC计数均小于127，ECU则可以返回主动错误状态；若TEC技术大于255，ECU进入总线关闭状态。
- **总线关闭（Bus Off）** ：节点因发送错误帧过多而被总线关闭，无法发送或接收消息。
	- 当被动错误状态下的通信节点，TEC大于255时，ECU进入总线关闭状态；
		- 总线关闭状态下，通信节点不能接受和发送消息；
		- CAN协议提供了总线关闭状态的恢复策略，即上层控制且通信节点检测到128次11个连续隐性后，ECU可以清零TEC和REC并重新进入主动错误状态。

### 2.2 错误计数器累加原则

说明：以下计数器累加原则，翻译自ISO-11898:2015 12.1.4.2 Error counting章节。

1. 当 **Receiver** 检测到错误时，接收REC应增 1，除非检测到的错误是在发送有效错误标志或过载标志时发生的位错误。
2. 当 **Receiver** 在发送错误标志后的第一位检测到一个显性位时，REC应递增 8。
3. 当 **Transmitter** 发送错误标志时，TEC应递增 8。
	- **例外 1** ：如果Transmitter处于 **被动错误状态** ，由于没有检测到显性ACK位而产生ACK错误，TEC不增加；如果Transmitter处于被动错误状态，发错被动错误标志时没有检测到显性位，TEC不增加。
		- **例外 2** ：如果Transmitter发送错误标志是因为在仲裁过程中发生了位错误，即发送隐性位，而回读到显性位，TEC不增加。
4. 如果 **Transmitter** 在发送主动错误标志或过载标志时检测到 **位错误** ，发送错误计数器应递增 8。
5. 如果 **Receiver** 在发送主动错误标志或过载标志时检测到 **位错误** ，则接收错误计数器应递增 8。
6. 任何节点在发送主动错误标志、被动错误标志或过载标志后，应最多容忍 7 个连续的主位。在检测到连续 14 个显性位后（在主动错误标志或过载标志的情况下），或在被动错误标志后检测到连续 8 个显性位后，以及在每个额外的连续 8 个显性位序列后，每个 **Transmitter** 应将其TEC递增 8，每个 **Receiver** 应将其REC递增 8。
7. 传输帧成功后（获得 ACK 且在 EOF 结束前未检测到错误），TEC应减 1，除非它已为 0。
8. 成功接收帧后（接收到 ACK 时隙前无错误且成功发送 ACK 位），如果REC在 1 和 127 之间，则应递减 1。如果REC为 0，则应保持为 0；如果大于 127，则应设置为 119 至 127 之间的值。

## 3 BUS OFF恢复策略

### 3.1 BUS OFF恢复

当通信节点进入BUS OFF状态后，需执行如下操作：

1. 通信节点的初始化CAN模块。
2. 通信节点在指定时间内禁止发送报文，处于只听状态。指定时间为快恢复时间或者慢恢复时间，参考3.2章节。
3. 通信节点恢复CAN正常通信。  
	![BUS OFF恢复过程](https://i-blog.csdnimg.cn/direct/a34d3cbe2e524a288dbf3d816fa90bdc.png#pic_center)

### 3.2 BUS OFF恢复策略

恢复策略选择

1. **自动恢复** ：ECU使用自动恢复功能，ECU检测到128次11个连续隐性位便可恢复通信。请参考节点状态图中Bus Off→Error Active的恢复策略。

2. **重启控制器** ：ECU不适用自动恢复功能，且不主动干预BUS OFF错误，则ECU无法自BUS OFF状态恢复总线通信。只能通过重新上电的方式，使节点进入主动错误状态。

3. **主动控制（快恢复和慢恢复）** ：ECU对于BUS OFF恢复状态进行编程控制，使用快恢复+慢恢复的策略，减少严重错误的ECU对于总线通信的干扰。主动控制是更为常见的方式。

恢复策略介绍

1. ECU进入BUS OFF状态后，会先执行 **快恢复** 操作。快恢复过程中，ECU会尝试向总线上发送报文（Tx Attempt阶段），若出现发送错误，TEC正常累加。TEC累加至255后，ECU再次进入BUS OFF状态。此时，ECU会在 *tBusOffQuick* 时间内，不再尝试向总线发送报文。等待 *tBusOffQuick* 时间后，ECU再次尝试向总线发送报文。

2. ECU在连续 **5次** 快恢复，即6次进入BUS OFF状态，将执行 **慢恢复** 操作。备注：具体次数由ECU设计决定，5次是常见值。

3. ECU执行 **慢恢复** ，会在进入BUS OFF状态等待 *tBusOffSlow* 时间后，再尝试向总线发送报文。

![BUS OFF恢复策略](https://i-blog.csdnimg.cn/direct/abbed43c30d0462ebe64c9550891188a.png#pic_center)

| 参数 | 最小值（ms） | 标称值（ms） | 最大值（ms） |
| --- | --- | --- | --- |
| *tBusOffQuick* | 45 | 50 | 70 |
| *tBusOffSlow* | 180 | 200 | 220 |

## 4 BUS OFF与DTC记录

1. 连续 *nBusOffCount* 次BUS OFF，且没有报文成功发送，则ECU需要记录BUS OFF的故障码。下表中值仅供参考，具体值需要参考OEM对于ECU的诊断设计。

| 参数 | 最小值 | 标称值 | 最大值 |
| --- | --- | --- | --- |
| *nBusOffCount* | \- | 8 | \- |

2. 当ECU进入BUS OFF模式后，只应存储BUS OFF故障码，不应存储节点丢失或通信超时故障码。因为BUS OFF状态无法接受总线报文，不能判定其他节点丢失或通信超时。

## 5 学习拓展

\[1\] [CAN控制器总线错误分析之CAN节点BusOff恢复过程分析与测试](https://news.eda365.com/appl/application/120011082481596.html/)  
\[2\] [利用Vector VH6501 CAN干扰仪经典CAN2.0/CANFD帧触发BUS OFF](https://blog.csdn.net/qfmzhu/article/details/107335763?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522171921025716800180631690%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=171921025716800180631690&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-4-107335763-null-null.142%5Ev100%5Epc_search_result_base4&utm_term=VH6501%20busoff&spm=1018.2226.3001.4187)