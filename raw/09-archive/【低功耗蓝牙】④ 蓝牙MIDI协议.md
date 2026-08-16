---
title: "【低功耗蓝牙】④ 蓝牙MIDI协议"
source: "https://www.bilibili.com/opus/621152876322443918/?from=readlist"
author:
published:
created: 2026-08-14
description:
tags:
  - "clippings"
---
## 摘要

蓝牙MIDI协议是智能乐器与计算机之间数据通信的协议格式，本文章主要介绍了通过低功耗蓝牙实现MIDI协议的步骤，以及MIDI指令的数据格式，最后介绍了使用ESP32蓝牙模块实现的一些电子乐器。

![](https://i0.hdslb.com/bfs/archive/bcaf2a0c395436277dafd82cec4fb6298e22dfe5.png@228w_128h_!web-dynamic.avif)

06:24

自制【电子】二胡，演奏二泉映月，附制作过程

8815

12

![](https://i0.hdslb.com/bfs/archive/e0fa1530a618012a5d848a32f7a91500aa91353a.jpg@228w_128h_!web-dynamic.avif)

06:32

自制卡林巴电子琴，可通过蓝牙连接手机库乐队 MIDI Boy【理科生的第一件乐器】

1.1万

23

## MIDI协议

MIDI 是 Musical Instrument Digital Interface 的简称，翻译成汉语为音乐数字接口。是20 世纪80 年代初为解决电声乐器之间的通信问题而提出的一种通信协议。协议传输的数据内容也主要是描述乐器的一些动作，比如钢琴上的按键按下或者抬起，按键按下的力度等等。

如下图所示，把电子琴连接到电脑上，就能在电脑上捕获电子琴演奏的动作：

![](https://i0.hdslb.com/bfs/article/ab3c8aa2de8ddd70ed69957742aea9c65e068d15.png@1192w.avif)

传统MIDI接口

传统的MIDI协议往往使用线缆实现，乐器又要经常移动，所以使用起来并不是很方便。苹果公司在2014年设计了支持低功耗蓝牙（BLE）的MIDI协议，用于乐器与智能手机/电脑通信。MMA(MIDI制造商成员协会)于2015年6月将该协议列为行业标准。智能手机普及以来，很多电子乐器中都加入了蓝牙MIDI的功能，可以通过蓝牙与手机或者平板电脑连接，从而实现智能教学。

![](https://i0.hdslb.com/bfs/article/02cd19999cd6427c2b3bc45991e968a3950c25cd.png@1192w.avif)

蓝牙MIDI接口

## 蓝牙MIDI协议

蓝牙MIDI协议是使用低功耗蓝牙GATT作为传输层，协议格式仍与有线MIDI协议一致。想要通过低功耗蓝牙设备实现MIDI协议，需满足如下两个要求：

一、在广播数据中广播128 Bit 的UUID：03B80E5A-EDE8-4B33-A751-6CE34EC4C700

![](https://i0.hdslb.com/bfs/article/c50871e41dbef730cdb8c62dbdc70275e75643aa.png@1192w.avif)

二、GATT中要包含一个UUID为03B80E5A-EDE8-4B33-A751-6CE34EC4C700的服务，该服务里面要有一个UUID为7772E5DB-3868-4112-A1A9-F2669D106BF3的特性，并且该特性的操作权限为 Read，Write，Notify。

![](https://i0.hdslb.com/bfs/article/b220490b62088cf3571dbd7ebb8092e874267823.png@1192w.avif)

蓝牙MIDI服务架构

蓝牙MIDI数据的交互，都是通过上述特性完成的。需要 **注意** 的是，一定要在广播数据中广播MIDI服务的UUID，才能被手机电脑等识别成MIDI设备。

## MIDI指令格式

![](https://i0.hdslb.com/bfs/article/f129d721e55d6b20f5c81f8dbfe54378ab5542e3.png@1192w.avif)

MIDI指令格式

如上图所示，一个MIDI指令包含5个字节，前两个字节表示时间戳，其中第一个字节的前两位必须是10，第二个字节的第一位必须是1（时间戳可不填，全零），所以前两个字节通常是0x80,0x80。第三个字节表示设备的动作，比如按键按下或抬起，音量调节等等。第四个字节和第五个字节是参数，其含义要根据第三个字节决定。

在MIDI指令中，每一个音符都有对应的编码，部分简朴音符与MIDI编码对应如下表：

![](https://i0.hdslb.com/bfs/article/f023030085df9937c9d72e693602bc1745207f75.png@1192w.avif)

上述表格中没有半音，下图以钢琴为例，给出了部分按键对应的MIDI编码：

![](https://i0.hdslb.com/bfs/article/7214533f6bb7f725be505ba7fd81cb912f6f457a.png@1192w.avif)

思考白键E5对应的MIDI编码是多少？

若以99的力度按下钢琴上的C4键，对应的MIDI指令是：

```javascript
80 80 90 48 63
```

上述指令的第三个字节90表示按键按下，第四个字节48表示钢琴上的键C4，最后一个字节63表示按下力度为0x63(十进制99)。

抬起钢琴上的C4键，对应的MIDI指令是：

```javascript
80 80 80 48 00
```

上述指令中第三个字节80表示按键抬起，第四字节48表示琴键C4，第五字节无意义。

更多MIDI指令的用法，请参考MIDI官网：https://www.midi.org

## 蓝牙MIDI协议实现代码

下面给出了基于ESP32 MicroPython平台的蓝牙MIDI设备代码，实现了一个按键，当按下按键的时候发送琴键按下指令，按键抬起的时候发送琴键抬起指令。

```python
from machine import Pin
from time import sleep_ms
import time
import ubluetooth #导入BLE功能模块

ble = ubluetooth.BLE()  #创建BLE设备
ble.active(True)  #打开BLE

MIDI_SERVER_UUID = ubluetooth.UUID(&amp;#39;03B80E5A-EDE8-4B33-A751-6CE34EC4C700&amp;#39;) 
MIDI_CHAR_UUID    = ubluetooth.UUID(&amp;#39;7772E5DB-3868-4112-A1A9-F2669D106BF3&amp;#39;) 

MIDI_CHAR = (MIDI_CHAR_UUID, ubluetooth.FLAG_READ | ubluetooth.FLAG_WRITE | ubluetooth.FLAG_NOTIFY , )
MIDI_SERVER = (MIDI_SERVER_UUID, (MIDI_CHAR ,  ) , ) #把MIDI特性放入MIDI服务

SERVICES = (MIDI_SERVER, )  #把MIDI服务放入服务集和中
((char_midi, ), ) = ble.gatts_register_services(SERVICES)  #注册服务到gatts

#设置BLE广播数据并开始广播
ble.gap_advertise(100, adv_data = b&amp;#39;\x02\x01\x05&amp;#39; 
                                + b&amp;#39;\x11\x07\x00\xC7\xC4\x4E\xE3\x6C\x51\xA7\x33\x4B\xE8\xEd\x5A\x0E\xB8\x03&amp;#39; 
                                + b&amp;#39;\x05\x09\x4D\x49\x44\x49&amp;#39;)

key1=Pin(0,Pin.IN,Pin.PULL_UP)
last_status = key1.value()  #记录上次按键的值
while True:
    if not key1.value() == last_status: #状态发生改变
      last_status = key1.value() #记录当前按键值

      if key1.value() == 0: #按键按下
        print(&amp;quot;Key Down&amp;quot;) #蓝牙连接后方能发送数据
        ble.gatts_notify(0, char_midi, b&amp;#39;\x80\x80\x90\x48\x63&amp;#39;)
 
      else:
        print(&amp;quot;Key UP&amp;quot;) #按键抬起
        ble.gatts_notify(0, char_midi, b&amp;#39;\x80\x80\x80\x48\x00&amp;#39;)
```

上述代码仅仅实现了一个按键，有兴趣的同学可以都实现几个按键，做成一个完整的乐器。

## 手机端APP

我们刚刚完成的代码仅仅完成了相关指令的发送，设备本身并不能发出声音，要想发出声音，需要手机或者电脑端软件的配合。手机端有很多APP支持蓝牙MIDI协议，iOS系统推荐库乐队 和 Echo Piano Lite ，Android系统推荐 泡泡钢琴，这些软件可通过下图二维码下载：

![](https://i0.hdslb.com/bfs/article/db7dcdc6aadac5ba5b38522fb535542506f7dba6.png@1192w.avif)

![](https://i0.hdslb.com/bfs/article/3fe36b28a95c9a14c093a008c81794ca141f0aa1.png@1192w.avif)

![](https://i0.hdslb.com/bfs/article/f2b2fc15fc842bc98c1884e0fd04f194b0e2045c.png@1192w.avif)

与对应的手机APP配合，就可以通过蓝牙硬件模拟智能乐器在手机上演奏了！
