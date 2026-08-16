---
title: "【低功耗蓝牙】⑤ HID协议"
source: "https://www.bilibili.com/opus/621571115942264766/?from=readlist"
author:
published:
created: 2026-08-14
description:
tags:
  - "clippings"
---
## 摘要

本文章主要介绍了蓝牙HID协议的实现方法，基于ESP32平台实现了蓝牙键盘，蓝牙鼠标，蓝牙自拍杆和蓝牙游戏手柄等设备，是初学者学习BLE HID协议很好的参考文章。

## HID设备

![](https://i0.hdslb.com/bfs/article/a41ca2bb012f25f1259d7d6e1c7a9f42d91b23c9.png@1192w.avif)

常见HID设备

HID（Human Interface Device）人体学接口设备，是生活中常见的输入设备，比如键盘鼠标游戏手柄等等。早期的HID是设备大部分都是通过USB接口来实现，蓝牙技术出现后，通过蓝牙作为传输层，实现了无线HID设备。通过低功耗蓝牙实现的HID功能一般简称为HOGP(HID over Gatt Profile)。

## 蓝牙HID设备的实现

任何低功耗蓝牙模块都可以通过软件开发实现HID功能，一个蓝牙模块要实现HID的功能，一般需满足如下两个条件：

**一、** 在广播数据中广播HID的UUID，设备外观，设备名称等信息。

![](https://i0.hdslb.com/bfs/article/77d14143705322d539e9f0aab76787458c658c1c.png@1192w.avif)

蓝牙HID广播数据

HID服务的UUID是0x1812，键盘的外观是0x03C1，鼠标的外观是0x03C2，游戏手柄的外观是0x03C3。

**二、** 在GATT中实现HID要求的服务和特性。

![](https://i0.hdslb.com/bfs/article/45bdcdc467ef57614aedbed02fdcd34a8e529829.png@1192w.avif)

蓝牙HIDS UUID

**0x1812** 是HID Service的UUID，必须要使用该UUID实现服务。

**0x2A4A** 是HID Information 的特性UUID，主要功能是展示HID的信息，其值为4个字节，前两个字节是HID版本，一般填入0x01，0x01，表示版本号为1.1。第三个字节是Country Code，一般填入0x00。第四个字节是HID Flags，一般填入0x02，表示Normally Connectable。

**0x2A4B** 是Report Map的特性UUID，主要功能是描述HID设备与HID主机数据交互的方式，即二者之间所发送的数据每一位的含义。

**0x2A4C** 是Control Point的特性UUID，该特性一定要可Write ，HID主机通过该特性告知HID设备主机的状态，比如电脑休眠后会告知蓝牙键盘也进入低功耗模式。

**0x2A4D** 是HID设备与HID主机之间交互数据(Report)的特性UUID。对于键盘设备，当某个按键按下时，HID设备发送数据到HID主机；当开关CapsLock，NumsLook和ScrollLook功能时，HID主机将相关指示灯的状态发送给HID键盘。所以键盘设备需要两个UUID为0x2A4D的特性，一个用于发送数据，另一个用于接收数据(使用UUID为0x2908的描述来区分)。而对于鼠标，游戏手柄这种只发送数据给电脑的设备，只需要定义一个0x2A4D的特性用来发送数据就可以了(当然定义两个也不会出错)。

**0x2A4E** 是协议模式的特性UUID，对于键盘和鼠标这两种设备，可能也会在电脑BIOS阶段使用，此阶段的计算机没有进入系统，难以支持复杂的设备。所以键盘鼠标就有两种模式，分别是Report模式和Boot模式，系统启动前使用的是Boot模式，HID设备与HID主机之间使用固定的数据格式进行交互。系统启动完成后，HID主机会通过该特性发数据给HID设备，通知HID设备切换成Report模式，在Report模式下，数据格式由 Report Map 决定。

该特性的数据值为0x00表示Boot模式，0x01表示Report模式。

在Boot模式下，Keyboard Input 的特性UUID是0x2A22，Output 的特性UUID是0x2A32；Mous Input 的特性UUID是0x2A33。

本文章只讨论Report模式，暂不讨论Boot模式！

## 蓝牙键盘

下列代码基于ESP32芯片MicroPython固件(V1.16以上)给出了简单的蓝牙键盘案例：

```python
from machine import Pin
import ubluetooth #导入BLE功能模块
from bluetooth import UUID

ble = ubluetooth.BLE()  #创建BLE设备
ble.active(True)  #打开BLE

ble.config(gap_name=&amp;quot;ESP Keyboard&amp;quot;)
ble.config(mtu=23)

HIDS = (                              # Service description: describes the service and how we communicate
    UUID(0x1812),                     # Human Interface Device
    (
        (UUID(0x2A4A), ubluetooth.FLAG_READ),       # HID information
        (UUID(0x2A4B), ubluetooth.FLAG_READ),       # HID report map
        (UUID(0x2A4C), ubluetooth.FLAG_WRITE),      # HID control point
        (UUID(0x2A4D), ubluetooth.FLAG_READ | ubluetooth.FLAG_NOTIFY, ((UUID(0x2908), 1),)),  # HID report / reference
        (UUID(0x2A4D), ubluetooth.FLAG_READ | ubluetooth.FLAG_WRITE,  ((UUID(0x2908), 1),)),  # HID report / reference
        (UUID(0x2A4E), ubluetooth.FLAG_READ | ubluetooth.FLAG_WRITE), # HID protocol mode
    ),
)

services = (HIDS,) 
handles = ble.gatts_register_services(services)

KEYBOARD_REPORT = bytes([    # Report Description: describes what we communicate
            0x05, 0x01,      # USAGE_PAGE (Generic Desktop)
            0x09, 0x06,      # USAGE (Keyboard)
            0xa1, 0x01,      # COLLECTION (Application)
            0x85, 0x01,      #     REPORT_ID (1)
            0x75, 0x01,      #     Report Size (1)
            0x95, 0x08,      #     Report Count (8)
            0x05, 0x07,      #     Usage Page (Key Codes)
            0x19, 0xE0,      #     Usage Minimum (224)
            0x29, 0xE7,      #     Usage Maximum (231)
            0x15, 0x00,      #     Logical Minimum (0)
            0x25, 0x01,      #     Logical Maximum (1)
            0x81, 0x02,      #     Input (Data, Variable, Absolute); Modifier byte
            0x95, 0x01,      #     Report Count (1)
            0x75, 0x08,      #     Report Size (8)
            0x81, 0x01,      #     Input (Constant); Reserved byte
            0x95, 0x05,      #     Report Count (5)
            0x75, 0x01,      #     Report Size (1)
            0x05, 0x08,      #     Usage Page (LEDs)
            0x19, 0x01,      #     Usage Minimum (1)
            0x29, 0x05,      #     Usage Maximum (5)
            0x91, 0x02,      #     Output (Data, Variable, Absolute); LED report
            0x95, 0x01,      #     Report Count (1)
            0x75, 0x03,      #     Report Size (3)
            0x91, 0x01,      #     Output (Constant); LED report padding
            0x95, 0x06,      #     Report Count (6)
            0x75, 0x08,      #     Report Size (8)
            0x15, 0x00,      #     Logical Minimum (0)
            0x25, 0x65,      #     Logical Maximum (101)
            0x05, 0x07,      #     Usage Page (Key Codes)
            0x19, 0x00,      #     Usage Minimum (0)
            0x29, 0x65,      #     Usage Maximum (101)
            0x81, 0x00,      #     Input (Data, Array); Key array (6 bytes)
            0xc0             # END_COLLECTION
        ])

#设置BLE广播数据并开始广播
ble.gap_advertise(100, adv_data = b&amp;#39;\x02\x01\x05&amp;#39;
                                + b&amp;#39;\x03\x03\x12\x18&amp;#39; #HID UUID
                                + b&amp;#39;\x03\x19\xC1\x03&amp;#39; #设备外观为键盘
                                + b&amp;#39;\x0D\x09&amp;#39; + &amp;quot;ESP Keyboard&amp;quot;.encode(&amp;quot;UTF-8&amp;quot;))

(h_info, h_map, _, h_repin, h_d1, h_repout, h_d2, h_model,) = handles[0]
# Write service characteristics
ble.gatts_write(h_info, b&amp;quot;\x01\x01\x00\x02&amp;quot;)     # HID info: ver=1.1, country=0, flags=normal
ble.gatts_write(h_map, KEYBOARD_REPORT)    # HID input report map
ble.gatts_write(h_d1, b&amp;quot;\x01\x01&amp;quot;)  # HID reference: id=1, type=input
ble.gatts_write(h_d2, b&amp;quot;\x01\x02&amp;quot;)  # HID reference: id=1, type=output
ble.gatts_write(h_model, b&amp;quot;\x01&amp;quot;)   # HID Protocol Model: 0=Boot Model, 1=Report Model

key = Pin(0,Pin.IN)#IO 0 用作按键
while True:
  if key.value() == 0:
    while key.value() == 0:
      pass
    ble.gatts_notify(0, h_repin, b&amp;#39;\x00\x00\x04\x00\x00\x00\x00\x00&amp;#39;)#按键A按下
    ble.gatts_notify(0, h_repin, b&amp;#39;\x00\x00\x00\x00\x00\x00\x00\x00&amp;#39;)#按键A抬起
```

上述代码实现了一个简单的蓝牙键盘，按下IO0对应的按键，发送按键A到HID主机。上述代码的ReportMap中定义了HID设备发给HID主机的数据为8个字节，定义如下图：

![](https://i0.hdslb.com/bfs/article/34c8b1808aedee4208cd2eee95049d0434e89357.png@1192w.avif)

Keyboard Report Data

第一个字节是Modifier按键，相应的位为1表示对应的按键按下(GUI在Windows下是Win键)；第二个字节保留，默认为零。第三到第八字节可表示六个按键，数据值为零表示无按键按下，按键对应的代码可搜索HID KeyCode，比如按键A对应的代码是0x04，按键B对应的代码是0x05，Enter键对应的代码是0x28。

比如：

0x80,0x00,0x06,0x00,0x00,0x00,0x00,0x00 表示同时按下Ctrl键和字母C键。

0x40,0x00,0x04,0x00,0x00,0x00,0x00,0x00 表示同时按下Shift键和字母A键。

需要 **注意** 的是，发送某个按键按下的数据后，需及时发送按键抬起的数据，否则系统会认为按键未抬起，从而触发长按操作。

## 蓝牙自拍杆

自拍杆是最近几年出现的产品，其实蓝牙自拍杆的本质就是一个超简易的蓝牙键盘。我们知道在手机拍照页面，音量调节按键可以触发快门拍照，蓝牙自拍杆本质上就是一个只能调节音量的蓝牙键盘。调节音量是键盘的高级功能，可通过如下代码试下：

```python
from machine import Pin
import ubluetooth #导入BLE功能模块
from bluetooth import UUID

ble = ubluetooth.BLE()  #创建BLE设备
ble.active(True)  #打开BLE

ble.config(gap_name=&amp;quot;ESP Keyboard&amp;quot;)
ble.config(mtu=23)

HIDS = (                              # Service description: describes the service and how we communicate
    UUID(0x1812),                     # Human Interface Device
    (
        (UUID(0x2A4A), ubluetooth.FLAG_READ),       # HID information
        (UUID(0x2A4B), ubluetooth.FLAG_READ),       # HID report map
        (UUID(0x2A4C), ubluetooth.FLAG_WRITE),      # HID control point
        (UUID(0x2A4D), ubluetooth.FLAG_READ | ubluetooth.FLAG_NOTIFY, ((UUID(0x2908), 1),)),  # HID report / reference
        (UUID(0x2A4D), ubluetooth.FLAG_READ | ubluetooth.FLAG_WRITE,  ((UUID(0x2908), 1),)),  # HID report / reference
        (UUID(0x2A4E), ubluetooth.FLAG_READ | ubluetooth.FLAG_WRITE), # HID protocol mode
    ),
)

services = (HIDS,) 
handles = ble.gatts_register_services(services)

MEDIA_REPORT = bytes([    # Report Description: describes what we communicate
            # Report ID 1: Advanced buttons
            0x05, 0x0C,       # Usage Page (Consumer)
            0x09, 0x01,       # Usage (Consumer Control)
            0xA1, 0x01,       # Collection (Application)
            0x85, 0x01,       #     Report Id (1)
            0x15, 0x00,       #     Logical minimum (0)
            0x25, 0x01,       #     Logical maximum (1)
            0x75, 0x01,       #     Report Size (1)
            0x95, 0x01,       #     Report Count (1)

            0x09, 0xCD,       #     Usage (Play/Pause)
            0x81, 0x06,       #     Input (Data,Value,Relative,Bit Field)
            0x0A, 0x83, 0x01, #     Usage (AL Consumer Control Configuration)
            0x81, 0x06,       #     Input (Data,Value,Relative,Bit Field)
            0x09, 0xB5,       #     Usage (Scan Next Track)
            0x81, 0x06,       #     Input (Data,Value,Relative,Bit Field)
            0x09, 0xB6,       #     Usage (Scan Previous Track)
            0x81, 0x06,       #     Input (Data,Value,Relative,Bit Field)
 
            0x09, 0xEA,       #     Usage (Volume Down)
            0x81, 0x06,       #     Input (Data,Value,Relative,Bit Field)
            0x09, 0xE9,       #     Usage (Volume Up)
            0x81, 0x06,       #     Input (Data,Value,Relative,Bit Field)
            0x0A, 0x25, 0x02, #     Usage (AC Forward)
            0x81, 0x06,       #     Input (Data,Value,Relative,Bit Field)
            0x0A, 0x24, 0x02, #     Usage (AC Back)
            0x81, 0x06,       #     Input (Data,Value,Relative,Bit Field)
            0xC0              # End Collection
        ])

#设置BLE广播数据并开始广播
ble.gap_advertise(100, adv_data = b&amp;#39;\x02\x01\x05&amp;#39;
                                + b&amp;#39;\x03\x03\x12\x18&amp;#39; #HID UUID
                                + b&amp;#39;\x03\x19\xC1\x03&amp;#39; #设备外观为键盘
                                + b&amp;#39;\x0D\x09&amp;#39; + &amp;quot;ESP Keyboard&amp;quot;.encode(&amp;quot;UTF-8&amp;quot;))

(h_info, h_map, _, h_repin, h_d1, h_repout, h_d2, h_model,) = handles[0]
# Write service characteristics
ble.gatts_write(h_info, b&amp;quot;\x01\x01\x00\x02&amp;quot;)     # HID info: ver=1.1, country=0, flags=normal
ble.gatts_write(h_map, MEDIA_REPORT)    # HID input report map
ble.gatts_write(h_d1, b&amp;quot;\x01\x01&amp;quot;)  # HID reference: id=1, type=input
ble.gatts_write(h_d2, b&amp;quot;\x01\x02&amp;quot;)  # HID reference: id=1, type=output
ble.gatts_write(h_model, b&amp;quot;\x01&amp;quot;)   # HID Protocol Model: 0=Boot Model, 1=Report Model

key = Pin(0,Pin.IN)#IO 0 用作按键
while True:
  if key.value() == 0:
    while key.value() == 0:
      pass
    ble.gatts_notify(0, h_repin, b&amp;#39;\x10&amp;#39;)#音量- 按下
    ble.gatts_notify(0, h_repin, b&amp;#39;\x00&amp;#39;)#音量- 抬起
```

上述代码试下了媒体控制的功能，按下IO0对应的按键，执行音量-的动作。上述Report Map中定义了键盘的高级按键，媒体控制的功能，发送给HID主机的数据为一个字节，每一位定义如下：

![](https://i0.hdslb.com/bfs/article/07cb196947f93e1f3488e9705ecf561f003f1123.png@1192w.avif)

Media Control 按键

0x10表示音量-，0x04表示下一曲。

## 蓝牙鼠标

下列代码给出了简单的蓝牙鼠标的示例代码：

```python
from machine import Pin
import ubluetooth #导入BLE功能模块
from bluetooth import UUID

ble = ubluetooth.BLE()  #创建BLE设备
ble.active(True)  #打开BLE

ble.config(gap_name=&amp;quot;ESP Mouse&amp;quot;)
ble.config(mtu=23)

HIDS = (                              # Service description: describes the service and how we communicate
    UUID(0x1812),                     # Human Interface Device
    (
        (UUID(0x2A4A), ubluetooth.FLAG_READ),       # HID information
        (UUID(0x2A4B), ubluetooth.FLAG_READ),       # HID report map
        (UUID(0x2A4C), ubluetooth.FLAG_WRITE),      # HID control point
        (UUID(0x2A4D), ubluetooth.FLAG_READ | ubluetooth.FLAG_NOTIFY, ((UUID(0x2908), 1),)),  # HID report / reference
        (UUID(0x2A4D), ubluetooth.FLAG_READ | ubluetooth.FLAG_WRITE,  ((UUID(0x2908), 1),)),  # HID report / reference
        (UUID(0x2A4E), ubluetooth.FLAG_READ | ubluetooth.FLAG_WRITE), # HID protocol mode
    ),
)

services = (HIDS,) 
handles = ble.gatts_register_services(services)

MOUSE_REPORT = bytes([    # Report Description: describes what we communicate
            0x05, 0x01,   # USAGE_PAGE (Generic Desktop)
            0x09, 0x02,   # USAGE (Mouse)
            0xa1, 0x01,   # COLLECTION (Application)
            0x85, 0x01,   #   REPORT_ID (1)
            0x09, 0x01,   #   USAGE (Pointer)
            0xa1, 0x00,   #   COLLECTION (Physical)
            0x05, 0x09,   #         Usage Page (Buttons)
            0x19, 0x01,   #         Usage Minimum (1)
            0x29, 0x03,   #         Usage Maximum (3)
            0x15, 0x00,   #         Logical Minimum (0)
            0x25, 0x01,   #         Logical Maximum (1)
            0x95, 0x03,   #         Report Count (3)
            0x75, 0x01,   #         Report Size (1)
            0x81, 0x02,   #         Input(Data, Variable, Absolute); 3 button bits
            0x95, 0x01,   #         Report Count(1)
            0x75, 0x05,   #         Report Size(5)
            0x81, 0x03,   #         Input(Constant);                 5 bit padding
            0x05, 0x01,   #         Usage Page (Generic Desktop)
            0x09, 0x30,   #         Usage (X)
            0x09, 0x31,   #         Usage (Y)
            0x09, 0x38,   #         Usage (Wheel)
            0x15, 0x81,   #         Logical Minimum (-127)
            0x25, 0x7F,   #         Logical Maximum (127)
            0x75, 0x08,   #         Report Size (8)
            0x95, 0x03,   #         Report Count (3)
            0x81, 0x06,   #         Input(Data, Variable, Relative); 3 position bytes (X,Y,Wheel)
            0xc0,         #   END_COLLECTION
            0xc0          # END_COLLECTION
        ])

#设置BLE广播数据并开始广播
ble.gap_advertise(100, adv_data = b&amp;#39;\x02\x01\x05&amp;#39;
                                + b&amp;#39;\x03\x03\x12\x18&amp;#39; #HID UUID
                                + b&amp;#39;\x03\x19\xC2\x03&amp;#39; #设备外观为鼠标
                                + b&amp;#39;\x0A\x09&amp;#39; + &amp;quot;ESP Mouse&amp;quot;.encode(&amp;quot;UTF-8&amp;quot;))

(h_info, h_map, _, h_repin, h_d1, h_repout, h_d2, h_model,) = handles[0]
# Write service characteristics
ble.gatts_write(h_info, b&amp;quot;\x01\x01\x00\x02&amp;quot;)     # HID info: ver=1.1, country=0, flags=normal
ble.gatts_write(h_map, MOUSE_REPORT)    # HID input report map
ble.gatts_write(h_d1, b&amp;quot;\x01\x01&amp;quot;)  # HID reference: id=1, type=input
ble.gatts_write(h_d2, b&amp;quot;\x01\x02&amp;quot;)  # HID reference: id=1, type=output
ble.gatts_write(h_model, b&amp;quot;\x01&amp;quot;)   # HID Protocol Model: 0=Boot Model, 1=Report Model

key = Pin(0,Pin.IN)#IO 0 用作按键
while True:
  if key.value() == 0:
    while key.value() == 0:
      pass
    ble.gatts_notify(0, h_repin, b&amp;#39;\x00\x0A\xF6\x00&amp;#39;)#X正方向和Y的负方向各移动10像素
    ble.gatts_notify(0, h_repin, b&amp;#39;\x00\x00\x00\x00&amp;#39;)#
```

上述代码实现了简单的蓝牙鼠标的功能，按下IO0对应的按键，鼠标指针将向右和向上各移动10个像素单位。上述代码中的Report Map定义了HID设备发给HID主机的数据为4个字节，每个字节含义如下：

![](https://i0.hdslb.com/bfs/article/19553201bd1855bec6a07809302f40011748f3f0.png@1192w.avif)

Mouse Report

一个标准的鼠标上面有三个按键(左键右键中间键)一个滑轮和一个光标。上述数据中，第一个字节的第三位表示三个按键按下与否(具体如何对应未知)，第二个字节表示鼠标指针在X方向的移动量，取值范围-127 ~ 127，第三字节表示鼠标指针在Y方向的移动量，取值范围是-127 ~ 127，第四个字节表示鼠标滑轮的滚动量，取值范围是-127 ~ 128。

比如：

0x01,0x00,0x00,0x00 表示鼠标上的某个按键按下。

0x00,0x09,0x09,0x00 表示鼠标指针向X正方向和Y正方向个移动9个像素单位。

0x00,0x00,0x00,0x01 表示鼠标滑轮滚动一格。

## 蓝牙游戏手柄

下列代码给出了简单的蓝牙游戏手柄案例：

```python
from machine import Pin
from time import sleep_ms
import ubluetooth #导入BLE功能模块
from bluetooth import UUID
import struct

ble = ubluetooth.BLE()  #创建BLE设备
ble.active(True)  #打开BLE

ble.config(gap_name=&amp;quot;ESP Joystick&amp;quot;)
ble.config(mtu=23)

HIDS = (                              # Service description: describes the service and how we communicate
    UUID(0x1812),                     # Human Interface Device
    (
        (UUID(0x2A4A), ubluetooth.FLAG_READ),       # HID information
        (UUID(0x2A4B), ubluetooth.FLAG_READ),       # HID report map
        (UUID(0x2A4C), ubluetooth.FLAG_WRITE),      # HID control point
        (UUID(0x2A4D), ubluetooth.FLAG_READ | ubluetooth.FLAG_NOTIFY, ((UUID(0x2908), 1),)),  # HID report / reference
        (UUID(0x2A4D), ubluetooth.FLAG_READ | ubluetooth.FLAG_WRITE,  ((UUID(0x2908), 1),)),  # HID report / reference
        (UUID(0x2A4E), ubluetooth.FLAG_READ | ubluetooth.FLAG_WRITE), # HID protocol mode
    ),
)

services = (HIDS,) 
handles = ble.gatts_register_services(services)

JOY_REPORT = bytes([    # Report Description: describes what we communicate
            0x05, 0x01,                    # USAGE_PAGE (Generic Desktop)
            0x09, 0x04,                    # USAGE (Joystick)
            0xa1, 0x01,                    # COLLECTION (Application)
            0x85, 0x01,                    #   REPORT_ID (1)
            0xa1, 0x00,                    #   COLLECTION (Physical)
            0x09, 0x30,                    #     USAGE (X)
            0x09, 0x31,                    #     USAGE (Y)
            0x15, 0x81,                    #     LOGICAL_MINIMUM (-127)
            0x25, 0x7f,                    #     LOGICAL_MAXIMUM (127)
            0x75, 0x08,                    #     REPORT_SIZE (8)
            0x95, 0x02,                    #     REPORT_COUNT (2)
            0x81, 0x02,                    #     INPUT (Data,Var,Abs)
            0x05, 0x09,                    #     USAGE_PAGE (Button)
            0x29, 0x08,                    #     USAGE_MAXIMUM (Button 8)
            0x19, 0x01,                    #     USAGE_MINIMUM (Button 1)
            0x95, 0x08,                    #     REPORT_COUNT (8)
            0x75, 0x01,                    #     REPORT_SIZE (1)
            0x25, 0x01,                    #     LOGICAL_MAXIMUM (1)
            0x15, 0x00,                    #     LOGICAL_MINIMUM (0)
            0x81, 0x02,                    #     Input (Data, Variable, Absolute)
            0xc0,                          #   END_COLLECTION
            0xc0                           # END_COLLECTION
        ])                  #         Report Size (1)

#设置BLE广播数据并开始广播
ble.gap_advertise(100, adv_data = b&amp;#39;\x02\x01\x05&amp;#39;
                                + b&amp;#39;\x03\x03\x12\x18&amp;#39; #HID UUID
                                + b&amp;#39;\x03\x19\xC3\x03&amp;#39; #设备外观为手柄
                                + b&amp;#39;\x0D\x09&amp;#39; + &amp;quot;ESP Joystick&amp;quot;.encode(&amp;quot;UTF-8&amp;quot;))

(h_info, h_map, _, h_repin, h_d1, h_repout, h_d2, h_model,) = handles[0]
# Write service characteristics
ble.gatts_write(h_info, b&amp;quot;\x01\x01\x00\x02&amp;quot;)     # HID info: ver=1.1, country=0, flags=normal
ble.gatts_write(h_map, JOY_REPORT)    # HID input report map
ble.gatts_write(h_d1, struct.pack(&amp;quot;&amp;lt;BB&amp;quot;, 1, 1))  # HID reference: id=1, type=input
ble.gatts_write(h_d2, struct.pack(&amp;quot;&amp;lt;BB&amp;quot;, 1, 2))  # HID reference: id=1, type=output
ble.gatts_write(h_model, b&amp;quot;\x01&amp;quot;) # HID Protocol Model: 0=Boot Model, 1=Report Model

key = Pin(0,Pin.IN)#IO 0 用作按键
while True:
  if key.value() == 0:
    while key.value() == 0:
      pass
    ble.gatts_notify(0, h_repin, b&amp;#39;\xf5\xf5\x00&amp;#39;)
    ble.gatts_notify(0, h_repin, b&amp;#39;\x00\x00\x00&amp;#39;)
```

上述代码实现了一个简单的蓝牙游戏手柄的功能，ReportMap中定义了HID设备发给HID主机的数据为3个字节，每个字节的含义如下;

![](https://i0.hdslb.com/bfs/article/dcb42c967feaa731e8d32fb34c046adec44fe80c.png@1192w.avif)

游戏手柄上一般由推杆和按键组成，上面数据的第一字节表示推杆在X方向的偏移量，范围是-127 ~ 127，第二字节表示推杆在Y方向的偏移量，范围是-127 ~ 128；第三字节表示是否有按键按下，最多支持八个按键。

## 上述代码中的BUG

一般的蓝牙设备在配对成功后，下次可以自动连接，上述代码运行后，需要在电脑或手机上删除该设备，再次连接才能生效，只有首次连接可以使用。断开连接后需要删除设备重新配对。

上述文档中的数据格式和取值范围和代码中的ReportMap一一对应，修改ReportMap后，数据格式和取值返回也要做对应的变化。

一般正常的蓝牙HID设备也要包含DIS(Device Information Service)设备信息服务，和BAS(Battery Service)电池电量服务，读者可自行添加。