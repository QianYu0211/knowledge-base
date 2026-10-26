---
title: "几种闪存技术：eMMC、UFS2.1、UFS3.0、SSD_ufs vs.emmcperformance scaling-CSDN博客"
source: "https://blog.csdn.net/qq_39111085/article/details/104229196?ops_request_misc=elastic_search_misc&request_id=a384ee335269faf4d3e3edc1f1fad724&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-5-104229196-null-null.142^v102^control&utm_term=emmc&spm=1018.2226.3001.4187"
author:
  - "[[成就一亿技术人!]]"
  - "[[hope_wisdom 发出的红包]]"
published:
created: 2026-08-19
description: "文章浏览阅读1.7w次，点赞5次，收藏84次。本文详细解析了SSD、eMMC和UFS三种存储技术的区别及发展历程，对比了NOR与NAND Flash的特点，介绍了eMMC的内部结构与优势，以及UFS的高速串行接口和全双工特性，最后对比了eMMC、UFS与SATA的速率。"
tags:
  - "clippings"
---
##### 它们的区别和作用

SSD 主要作用是取代 PC/服务器 上的 HDD 硬盘，它需要：

- 超大容量（百GB~TB级别）
- 极高的并行性以提高性能
- 对功耗，体积等要求并不敏感
- 兼容已有接口技术 （如SATA，PCI）

SSD发展历程： [https://blog.csdn.net/u012414189/article/details/85063550](https://blog.csdn.net/u012414189/article/details/85063550)

而 eMMC 和 UFS主要是针对移动设备发明的，它们需要：

- 适当的容量、性能
- 对功耗 ，体积的要求极其敏感
- 仅需遵循一定的接口标准

##### eMMC的发展历程

MMC，是一种闪存卡 (Flash Memory Card) 标准，它定义了MMC的架构以及访问Flash Memory的接口和协议。而eMMC（embedded MultiMediaCard）则是对MMC的一个拓展，以满足更高标准的性能、成本、体积、稳定、易用等的需求。

Flash分为两种规格： **NOR Flash和NAND Flash** ，两者均为非易失性闪存模块。  
1988年，Intel首次发出NOR flash技术,彻底改变了原先由EPROM和EEPROM一统天下的局面。NOR类似于DRAM, 以存储程序代码为主，可以让微处理器直接读取。因为读取速度较快，晶片容量较低，所以多应用在手机这样的通讯产品中。  
1989年，东芝公司发表NAND flash结构，强调降低每比特的成本,更高的性能，并且像磁盘一样可以通过接口轻松升级。因为NAND flash的晶片容量相对于NOR大，更像硬盘，写入与清除资料的速度远快于NOR，所以多应用在小型机以储存资料为主。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9ee65a29ce41877054ddb6da1e260442.png)

NAND Flash的存储单元发展：从 **SLC, MLC到TLC，超越摩尔定律**  
TLC (Triple-Level Cell), 即3bit/cell，速度慢，寿命短，价格便宜，约500次读写寿命。  
MLC (Multi-Level Cell), 即2bit/cell,速度一般，寿命一般，价格一般，约3000-10000次读写寿命。  
SLC (Single-Level Cell), 即1bit/cell，读写速度快，寿命长，价格是MLC三倍以上，约10万次读写寿命。

**摩尔定律** 是由英特尔（Intel）创始人之一戈登·摩尔（Gordon Moore）提出来的。其内容为： **当价格不变时，集成电路上可容纳的晶体管数目，约每隔18个月便会增加一倍，性能也将提升一倍。换言之，每一美元所能买到的电脑性能，将每隔18个月翻两倍以上。而NAND Flash行业的摩尔定律周期则只有12个月。**

随着纳米制程工艺和存储单元的发展，使得同样大小的芯片有更高密度和更多的存储单元，Flash得以在容量迅速增加的同时，还大幅降低了单位存储容量的成本，嵌入式存储eMMC即营运而生。

**eMMC ( Embedded Multi Media Card)** 采用统一的MMC标准接口， 把高密度NAND Flash以及MMC Controller封装在一颗BGA芯片中。针对Flash的特性，产品内部已经包含了Flash管理技术，包括错误探测和纠正，flash平均擦写，坏块管理，掉电保护等技术。用户无需担心产品内部flash晶圆制程和工艺的变化，同时eMMC单颗芯片为主板内部节省更多的空间。

##### 从eMMC到UFS

eMMC的协议因为多年未更新，并且eMMC的数据时序标准到达瓶颈，所以很难再有新的发展。而且并行8总线的eMMC因为需要并行-串行转换，速率很难提升上去。而连接简单、高速率、串行方式的UFS变成新的趋势。

UFS (Universal Flash Storage，通用闪存存储)，UFS是一种高性能接口，设计用于需要最小化功耗的应用，包括智能手机和平板电脑等移动系统以及汽车应用，其高速串行接口和优化协议可显着提高吞吐量和系统性能。

###### 数据传输形式不同

- eMMC为半双工结构，同一时间下只能读取或者写入，但是UFS是全双工结构，既可以读取又可以写入。
- UFS使用的是差分传输结构，而eMMC使用的是单线传输形式需要考虑噪声容限等指标，且信号电平要高于UFS，因此速率受限不如UFS。

###### eMMC、UFS、SATA速率比较

eMMC与UFS相比，总线数量太多，但最终处理都是要变为串行机器码的，所以如果eMMC协议没有革新，则迟早被UFS取代。况且因为温度导致的时序飘移等情况，在eMMC5.1版本下，为了优化在HS400模式下这个问题，专门增加了strobe这个引脚来自适应时序对其，如果eMMC再升高传输模式，则就需要做出新的改变。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ed467106adbd71e39f8264178e55c458.png)  

###### 总线传输形式不同

虽然都是同步传输方式，即都需要时钟信号作为参考，但：

- eMMC使用8条数据总线做复用（地址和数据，发送和接收），外加命令线以及新增的STROBE线；需要的IO口很多，比较复杂
- UFS只需要两组差分线，一条是时钟线就可以了，并且是双向数据发送接收，全双工；IO口配置也比较简单，就像USB3.0一样  
	![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/78628d95d4e947e24dd7af3bf91d0526.png)  
	  
	![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/27bbb14fdc028d24a8cc4a0d5d3f1cdf.png)

##### UFS2.1到UFS3.0

UFS常见的标准有UFS2.0、UFS2.1，加上最新的UFS3.0。3.0是在2018年初由固态技术协会（JEDEC）正式发布的，采用了全新的HS-G4规范，一年多后，UFS3.0才大量上市。

- 全新HG-G4规范，带宽翻倍，UFS3.0闪存的单通道带宽提升到11.6Gbps，比之前G3标准的UFS2.1翻了一倍，考虑到UFS3.0支持双通道读写，实际的带宽为11.6\*2=23.2Gbps，换算过来就是2.9GB/s。  
	![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c9386f84ad4d74ac9667507648909815.jpeg)
- 高密度的存储器，UFS3.0采用了全新的NAND存储器，降低功耗的同时还提升性能。  
	UFS3.0接口带宽高达2900MB/s  
	相比UFS2.1的1450MB/s，性能翻倍  
	UFS 3.0的接口电压比UFS2.1更低  
	给闪存颗粒供电的VCC电源从2.9V降到了2.5V  
	给主控供电的VCCQ从1.8V降到了1.2V  
	接口电压的降低直接关系功耗和发热  
	UFS3.0每GB/s顺序读取功耗比UFS2.1降低了60%  
	UFS3.0每GB/s顺序写入功耗比UFS2.1降低了48%  
	![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/348aa04cdc17ef60ebc17077e6fa87dd.jpeg)

##### 支持Write Turbo的UFS3.0

UFS3.0的性能瓶颈是写入速度，Write Turbo依靠SLC动态加速，当Host往闪存写入数据时候，首先将数据写入速度更快的SLC，当用户不再需要写入数据时，UFS主控会将SLC的数据写入速度较慢的TLC，类似于PC CPU缓存和内存间的关系。  
2020年安卓高端旗舰手机标配：  
**骁龙865+LPDDR5+Write Turbo UFS3.0**  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/269d3ee07a716bee95f1abeec71d169f.jpeg)  
参考链接： [https://blog.csdn.net/LUOHUATINGYUSHENG/article/details/94624348](https://blog.csdn.net/LUOHUATINGYUSHENG/article/details/94624348)  
[https://zhuanlan.zhihu.com/p/26551438](https://zhuanlan.zhihu.com/p/26551438)