---
title: MQTT协议快速入门
source: https://hd1611756908.github.io/2024/05/05/MQTT%E5%8D%8F%E8%AE%AE%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/
author:
published: 2024-05-05
created: 2026-08-12
description: "123456作者: 夜泊1990企鹅: 1611756908鹅群: 948233848邮箱: hd1611756908@163.com博客: https://hd1611756908.github.io/B 站: https://space.bilibili.com/514155929/  第一章 MQTT协议概述第1节 MQTT协议介绍1MQTT是一种基于发布/订阅模式的轻量级消息传输协议，设计"
tags:
  - clippings
---
## 第一章 MQTT协议概述

### 第1节 MQTT协议介绍

```
MQTT是一种基于发布/订阅模式的轻量级消息传输协议，设计用于低带宽、高延迟或不稳定的网络环境，广泛应用于物联网领域
```

### 第2节 MQTT协议版本更迭

```xml
1999年    MQTT v1.0 由IBM Andy Stanford-Clark和Eurotech的Arlen Nipper为石油管道远程监控项目发明的
2010年    MQTT v3.1 IBM将 MQTT v3.1版本协议开放给所有人使用
2014年10月 MQTT v3.1.1 MQTT正式成为OASIS标准                   --当前主流版本
2019年03月 OASIS发布了 MQTT v5.0 5.0版本成为了OASIS的新的标准     -- 后起之秀版本
```

### 第3节 MQTT协议的应用场景

```xml
1. 智能家居       : MQTT可以用于连接各种家电设备和传感器，实现设备之间的通信和控制
2. 车联网         : MQTT可以用于车辆与车辆、车辆与基础设施之间的实时通信
3. 工业物联网     : MQTT可以用于连接各种工业设备和传感器，实现设备的监测和控制
4. 远程监控和控制  : MQTT可以用于将传感器数据发布到云平台，供其他设备或应用程序订阅和使用
5. 消息通知       : MQTT可以用于提供实时消息通知功能
6. 资源监控与管理  : MQTT能够提供对物联网设备的实时监控和管理功能
7. 数据采集和分析  : MQTT也可以用于数据采集和分析
```

### 第4节 MQTT协议优势

```
1. 常见的计算机语言(C/C++、Java、Python、Go...)都有支持MQTT协议的客户端
2. MQTT协议是建立在TCP/IP协议基础之上,所以MQTT协议安全可靠
3. 服务质量设置，MQTT协议提供了三种服务质量配置分别为:
    Qos 0: 消息可能丢失 
    Qos 1: 消息不会丢失,但是可能重复
    Qos 2: 消息不会丢失也不会重复
4. 心跳保活: 由于网络问题可能造成连接陷于假死状态,为了判断客户端和代理是否出现异常，MQTT定义自己的心跳机制，定期向代理发送报文，以便于快速识别出异常连接，让客户端快速与代理断开连接
5. 持久会话: 代理保留客户端发送过来的消息，以便于消息订阅端上线立刻获取消息
....
```

### 第5节 MQTT协议报文

#### 5.1 报文(数据包)结构

```
MQTT报文(数据包)由三部分组成:
1. 固定报头(Fixed header)    : 所有数据包中都包含此报头,用来表示数据包的类型,以及数据包的分组累标识
2. 可变报头(Variable header) : 存在于部分MQTT数据包中,数据包类型决定了可变头是否存在及其具体内容
3. 有效载荷(Payload)         : 存在于部分MQTT数据包中,表示客户端收到的具体内容
```

![baowen](https://note.youdao.com/yws/api/personal/file/WEB1cfb82320eec5fcc9bb37ccb79992517?method=download&shareKey=d1b7ba2a4ab5fd51216da8d8e69f8555)

#### 5.2 固定报头格式(Fixed header)

![mqtt](https://note.youdao.com/yws/api/personal/file/WEB25ef81539ff6766878d259f26bd9ef68?method=download&shareKey=b2de947188cdf5039cf1c24f6f71aae2)

##### 5.2.1 固定报头介绍

```
首先固定报头一定存在所有的数据包中,也就是说在客户端与服务器端进行通信时,传递的所有消息中都包含固定报头
那么固定报头主要是做什么的呢?
举例说明：假如客户端向服务器端发送一个请求，那么问题来了，服务器端怎么判断客户端发送的这个请求是什么类型的请求，要给服务器传递数据呀，还是只是来判断服务器是否存活，还是想和服务器断开连接的请求，所以固定报头里面就保存了客户端向服务器端发送请求的类型，也就是上图中所谓的MQTT报文类型(占据byte1的高4位). byte1还有低4位，取值范围也是0-15，但是基本都是保留位,只有当MQTT的高4位类型是PUBLISH的时候，byte1的低4位取值才有意义,下面会详细介绍
```

##### 5.2.2 固定报头格式介绍

> byte1 高4位 MQTT报文类型介绍

```
byte1的高4位取值范围:
二进制: 0000~1111
十进制: 0-15

具体每一个值对应的格式详细如下
```

| 报文类型名称 | 报文类型对应值 | 报文表示的意义 | 描述 | 备注 |
| --- | --- | --- | --- | --- |
| Reserved | 0 | 禁止 | 保留位 | 默认值 高4位 取值 0000 |
| CONNECT | 1 | 客户端到服务端 | 客户端请求连接服务端 | 高4位 取值 0001 |
| CONNACK | 2 | 服务端到客户端 | 连接报文确认 | 高4位 取值 0010 |
| PUBLISH | 3 | 两个方向都允许 | 发布消息 | 高4位 取值 0011 |
| PUBACK | 4 | 两个方向都允许 | QoS 1消息发布收到确认 | 高4位 取值 0100 |
| PUBREC | 5 | 两个方向都允许 | 发布收到（保证交付第一步） | 高4位 取值 0101 |
| PUBREL | 6 | 两个方向都允许 | 发布释放（保证交付第二步） | 高4位 取值 0110 |
| PUBCOMP | 7 | 两个方向都允许 | QoS 2消息发布完成（保证交互第三步） | 高4位 取值 0111 |
| SUBSCRIBE | 8 | 客户端到服务端 | 客户端订阅请求 | 高4位 取值 1000 |
| SUBACK | 9 | 服务端到客户端 | 订阅请求报文确认 | 高4位 取值 1001 |
| UNSUBSCRIBE | 10 | 客户端到服务端 | 客户端取消订阅请求 | 高4位 取值 1010 |
| UNSUBACK | 11 | 服务端到客户端 | 取消订阅报文确认 | 高4位 取值 1011 |
| PINGREQ | 12 | 客户端到服务端 | 心跳请求 | 高4位 取值 1100 |
| PINGRESP | 13 | 服务端到客户端 | 心跳请求 | 高4位 取值 1101 |
| DISCONNECT | 14 | 客户端到服务端 | 客户端断开连接 | 高4位 取值 1110 |
| Reserved | 15 | 禁止 | 保留位 | 高4位 取值 1111 |

> byte1 低4位MQTT其他配置介绍

```
byte1 低4位默认大多都是保留位,没有意义,但是当MQTT类型为PUBLISH(0011)时 低4位开始有意义,具体如下:
```

![geshi](https://note.youdao.com/yws/api/personal/file/WEB1fdb1211e214cb9c03493c1e267db436?method=download&shareKey=55700ec7529325c88727738673944231)

```
当byte1的高4位取值为0011时，byte1的低4位就会有意义

DUP位: 如果设置0客户端或服务端第一次请求发送这个PUBLISH报文,如果设置1，表示这可能是一个早前报文请求的重发
Qos位: 由2位组成取值范围是0-3(00~11) 服务质量,每一个值代表的意义如下

Qos值    bit2    bit1      描述
 0        0       0     消息可能丢失
 1        0       1     消息不会丢失,但是可能重复
 2        1       0     消息不会丢失,也不会重复
 x        1       1     保留位
 
RETAIN位: 1/0
    1. 如果设置1 那么服务器端必须保存Qos信息,以便将Qos信息传递给其它的订阅者；
    2. 如果服务器端收到一个RETAIN为1并且QoS0的消息，服务器端需要丢弃掉原来那个主题保留任何消息，将这个新的QoS0消息当作主题的新保留消息 
    3. 服务端发送消息给订阅者,如果订阅者是一个新的，那么RETAIN为1，如果订阅者是一个已经存在的RETAIN为0
    4. RETAIN为0并且有效载荷为0的报文，服务端会当作正常消息处理
```

#### 5.3 可变报头

```
可变报头包含主题名和报文标识符
```

#### 5.4 有效载荷

```
主要是传递的消息
```

### 第6节 MQTT协议的工作原理

![mqtt](https://note.youdao.com/yws/api/personal/file/WEB9bee4853a8bd940dbc1927ddfc1ba49c?method=download&shareKey=f7c7d216b16a34aa5c478606e1c1aed5)

```
MQTT协议基于TCP/IP协议,TCP/IP协议是一个安全稳定的协议,通信需要服务端和客户端经历三次握手四次挥手，建立一个稳定的通道然后在进行数据传输。MQTT协议建立在TCP/IP协议之上,也是需要编写服务端(上图中的Broker)和客户端(消息发布者和消息订阅者).

下面就用案例进行MQTT学习
```

## 第二章 MQTT代理服务器介绍和搭建

### 第1节 MQTT代理服务器介绍

#### 1.1 Eclipse Mosquitto

```
官网地址: https://mosquitto.org/

介绍:
1. 开源的消息代理服务器
2. 支持MQTT协议版本 3.1 3.1.1 5.0 版本
3. 轻量级,适用从低功耗单板计算机到完整服务器的所有设备
4. 不支持集群功能
```

#### 1.2 EMQX

```
官网地址: https://www.emqx.io/zh

介绍:
1. 开源大规模分布式MQTT代理服务器
2. 支持MQTT协议版本 3.1 3.1.1 5.0 版本
3. 单台并发连接数可以高达一亿,每秒处理百万级消息
4. 安全可靠的消息传递
```

### 第2节 MQTT代理服务器安装(EMQX)

```
EMQX安装: windows
1. 下载地址: https://www.emqx.io/zh/get-started
2. 安装步骤:
    第一步: 下载 emqx-5.3.2-windows-amd64.zip 安装包,版本可能和我这个不同
    第二步: 解压
    第三步: 打开命令行(以管理员身份运行),切换到解压目录的bin目录下
    第四步: 安装,在bin目录下执行EMQX安装命令  emqx.cmd install ,完成之后有类似下面的输出,说明安装成功,只需要安装一次
        D:\app\emqx-5.3.2-windows-amd64\bin>emqx.cmd install
        EMQX_NODE__DB_ROLE [node.role]: core
        EMQX_NODE__DB_BACKEND [node.db_backend]: mnesia
        D:\app\emqx-5.3.2-windows-amd64\erts-13.2.2.4\bin\erlsrv.exe: Service emqx_5.3.2 added to system.
        [SC] ChangeServiceConfig 成功
    第五步(可选择): 如果想将EMQX从windows上卸载可以执行 emqx.cmd uninstall 命令
    第六步: 去windows服务列表中找到第四步安装的EMQX的服务,鼠标右键启动
    第七步: 在命令行输入 emqx.cmd console 命令，查看是否启动成功,如果有类似以下日志启动成功
        D:\app\emqx-5.3.2-windows-amd64\bin>emqx.cmd console
        EMQX_LOG__CONSOLE_HANDLER__ENABLE [log.console.enable]: true
        EMQX_NODE__DB_ROLE [node.role]: core
        EMQX_NODE__DB_BACKEND [node.db_backend]: mnesia

        D:\app\emqx-5.3.2-windows-amd64>D:\app\emqx-5.3.2-windows-amd64\erts-13.2.2.4\bin\erl.exe -mode embedded -boot "D:\app\emqx-5.3.2-windows-amd64\releases\5.3.2\start" -config "D:\app\emqx-5.3.2-windows-amd64\data\configs\app.2024.05.06.16.38.19.config" -args_file "D:\app\emqx-5.3.2-windows-amd64\data\configs\vm.2024.05.06.16.38.19.args" -mnesia dir 'd:/app/emqx-5.3.2-windows-amd64/data/mnesia/emqx@127.0.0.1'
        Listener ssl:default on 0.0.0.0:8883 started.
        Listener tcp:default on 0.0.0.0:1883 started.
        Listener ws:default on 0.0.0.0:8083 started.
        Listener wss:default on 0.0.0.0:8084 started.
        Listener http:dashboard on :18083 started.
        EMQX 5.3.2 is running now!
        Eshell V13.2.2.4  (abort with ^G)
        v5.3.2(emqx@127.0.0.1)1>
    
    第八步:通过浏览器访问控制台http://127.0.0.1:18083 默认初始化用户名: admin 默认密码: public 进入之后会让你重新修改密码
    
    
注意事项: (第六步+第七步)这种启动方式在开发时使用,如果想正式环境使用请遵循官网命令启动介绍: 正式环境启动在bin目录下直接输入 emqx start进行EMQX启动,这时不需要（第六步和第七步）

官网命令详细使用地址: https://www.emqx.io/docs/zh/latest/admin/cli.html
```

### 第3节 MQTT客户端工具介绍

```
EMQX官网自带工具MQTTX官网地址: https://mqttx.app/zh/downloads
```

### 第4节 MQTT客户端工具安装和使用

```
傻瓜式安装,无脑下一步
```

## 第三章MQTT协议练习(Java版)

```
首先在编写代码前需要安装Broker,如果已经安装完,需要将Broker启动起来,我这里使用EMQX作为Broker. 接下来开始代码编写.

基于maven项目,需要向项目中添加maven依赖,依赖来自于EMQX官网地址为: https://www.emqx.com/zh/mqtt-client-sdk
<dependency>
    <groupId>org.eclipse.paho</groupId>
    <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
    <version>1.2.2</version>
</dependency>

依赖添加完成之后,需要编写 客户端-消息订阅者和 客户端-消息发布者
```

### 第1节 MQTT客户端之消息订阅者

```java
import org.eclipse.paho.client.mqttv3.*;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;

/**
 * 消息订阅者
 */
public class Subscribe {

    //代理服务器地址
    private static final String BROKER="tcp://127.0.0.1:1883";
    //客户端ID
    private static final String CLIENT_ID="test01";

    public static void main(String[] args) throws MqttException {

        /*
            创建消息发布客户端对象
            第一个参数: 代理服务器地址
            第二个参数: 给客户端起一个唯一名字
            第三个参数: 使用MemoryPersistence进行动态消息存储,如果不给值,使用内部默认的类进行持久消息存储
         */
        MqttClient client = new MqttClient(BROKER, CLIENT_ID, new MemoryPersistence());

        //创建MQTT连接配置
        MqttConnectOptions connection = new MqttConnectOptions();
        /*
            是否清空会话session
            如果设置false那么就会保留历史session
            如果设置成true每次连接都是新的session
        */
        connection.setCleanSession(true);

        //将客户端和连接关联
        client.connect(connection);

        //设置回调函数(当发布者发布消息后被订阅者监听到,并且通过回调函数进行接收)
        client.setCallback(new MqttCallback() {
            //连接丢失
            @Override
            public void connectionLost(Throwable throwable) {
                System.out.println("连接已丢失..."+throwable.getMessage());
            }

            // 消息已送达
            @Override
            public void messageArrived(String s, MqttMessage mqttMessage) throws Exception {
                System.out.println("消息已送达...订阅的主题:"+s+" 收到的消息为:\n"+new String(mqttMessage.getPayload()));
            }

            // 消息接收完成
            @Override
            public void deliveryComplete(IMqttDeliveryToken iMqttDeliveryToken) {
                System.out.println("消息接受已完成...."+iMqttDeliveryToken.isComplete());
            }
        });

        /*
         * 订阅test主题的消息
         * subscribe消息订阅函数
         * 参数1: 要订阅的主题
         * 参数2: 服务质量,取值为 0、1、2
         */
        client.subscribe("test",2);
        System.out.println("消息监听开始.....");
    }
}
```

### 第2节 MQTT客户端之消息发布者

```java
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;

import java.io.UnsupportedEncodingException;
import java.nio.charset.StandardCharsets;

/**
 * 消息发布者
 */
public class Publish {
    //代理服务器地址
    private static final String BROKER="tcp://127.0.0.1:1883";
    //客户端ID
    private static final String CLIENT_ID="test02";
    public static void main(String[] args) throws MqttException, UnsupportedEncodingException {

        /*
            创建消息发布客户端对象
            第一个参数: 代理服务器地址
            第二个参数: 给客户端起一个唯一名字
            第三个参数: 使用MemoryPersistence进行动态消息存储,如果不给值,使用内部默认的类进行持久消息存储
         */
        MqttClient client = new MqttClient(BROKER, CLIENT_ID, new MemoryPersistence());

        //创建MQTT连接配置
        MqttConnectOptions connection = new MqttConnectOptions();
        /*
            是否清空会话session
            如果设置false那么就会保留历史session
            如果设置成true每次连接都是新的session
        */
        connection.setCleanSession(true);
        //将客户端和连接关联
        client.connect(connection);

        /*
            创建消息
            调用有参构造函数进行创建,参数传一个byte数组,数组中是要发布的消息(有效载荷)
         */
        MqttMessage message = new MqttMessage("我是要发布的消息".getBytes(StandardCharsets.UTF_8));
        //设置服务质量
        message.setQos(2);
        /*
            消息发布
            第一个参数: 消息发送到哪一个主题
            第二个参数: 发送的消息对象
         */
        client.publish("test",message);

        //断开连接
        client.disconnect();
        //关闭
        client.close();
    }
}
```