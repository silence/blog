本文为 《图解 TCP/IP》的重点摘抄

[TOC]

### 第 1 章 网络基础知识

![image-20210620183402440](https://raw.githubusercontent.com/silence/blog/assets/assets/20210620183402.png)

##### 1.2.4 计算机网络的产生

20 世纪八十年代

![image-20210620193659876](https://raw.githubusercontent.com/silence/blog/assets/assets/20210620193659.png)

##### 1.5.4 OSI 参考模型中的各个分层的作用

![image-20210620195916167](https://raw.githubusercontent.com/silence/blog/assets/assets/20210620195916.png)

##### 1.6.1 7 层通信

![image-20210620200104334](https://raw.githubusercontent.com/silence/blog/assets/assets/20210620200104.png)

##### 1.8.2 地址的层次性

MAC 寻址参考的表叫做地址转发表，而 IP 寻址中所参考的叫做路由控制表。

#### 1.9 网络的构成要素

![image-20210622114523487](https://raw.githubusercontent.com/silence/blog/assets/assets/20210622114523.png)

表 1.4 总结了各种不同的数据链路、通信媒介及其标准传输速率。

![image-20210622114744088](https://raw.githubusercontent.com/silence/blog/assets/assets/20210622114744.png)

> 传输速率高指的不是单位数据流动的速度有多快，而是指**单位时间内传输的数据量有多少**。

图 1.49 各种设备及其对应网络分层概览。

![image-20210622115744551](/Users/leereliu/Library/Application Support/typora-user-images/image-20210622115744551.png)

### 第 2 章 TCP/IP 基础知识

#### 2.4 TCP/IP 协议分层模型

##### 2.4.1 TCP/IP 与 OSI 参考模型

![image-20210623231139454](https://raw.githubusercontent.com/silence/blog/assets/assets/20210623231139.png)

图 2.18 TCP/IP 各层对邮件的收发处理

![image-20210623232133442](https://raw.githubusercontent.com/silence/blog/assets/assets/20210623232133.png)

### 第 3 章 数据链路

MAC 地址长 48 比特，结构如图 3.5 所示。在使用网卡（NIC）的情况下，MAC 地址一般会被烧入到 ROM 中。因此，任何一个网卡的 MAC 地址都是唯一的，在全世界都不会有重复。

![image-20210701000033951](https://raw.githubusercontent.com/silence/blog/assets/assets/20210701000034.png)

HDMI (High-Definition Multimedia Interface)，高清晰度多媒体接口。

### 第 4 章 IP 协议

网络层的主要作用是“实现终端节点之间的通信”。这种终端节点之间的通信也叫“点对点 (end-to-end) 通信”。

网络层的下一层 --- 数据链路层的主要作用是在**互连同一种数据链路的节点之间进行包传递。**而一旦跨越了多种数据链路，就需要借助网络层。网络层可以跨越不同的数据链路，即使是在不同的数据链路上也能实现两端节点之间的数据包传输。

![image-20210701002155482](https://raw.githubusercontent.com/silence/blog/assets/assets/20210701002155.png)

> **主机**的定义是指“配置有 IP 地址，但是不进行路由控制的设备”。既配有 IP 地址又具有路由控制能力的设备叫做“**路由器**”，而节点则是主机和路由器的统称。

![image-20210701002429956](https://raw.githubusercontent.com/silence/blog/assets/assets/20210701002429.png)

#### 4.2 IP 基础知识

IP 大致分为三大作用模块，它们是 **IP 寻址**、**路由**（最终节点为止的转发）以及 **IP 分包与组包**。

##### 4.2.2 路由控制

路由控制是指将分组数据发送到最终目标地址的功能。一个数据包之所以能够成功地到达最终的目标地址，全靠路由控制。

这里可以用快递的送货方式来打比方。IP 数据包犹如包裹，而送货车犹如数据链路。包裹不可能自己移动，必须有送货车承载转运。而一辆送货车只能将包裹送到某个区间范围内。**每个不同区间的包裹将由对应的送货车承载、运输**。IP 的工作原理也是如此。

![image-20210701003622522](https://raw.githubusercontent.com/silence/blog/assets/assets/20210701003622.png)

为了将数据包发给目标主机，所有主机都维护着一张路由控制表。**该表记录 IP 数据在下一步应该发给哪个路由器**。IP 包将根据这个路由表在各个数据链路上传输。

##### 4.2.4 IP 属于面向无连接型

#### 4.3 IP 地址的基础知识

32 位正整数，以每 8 位一组，分成 4 组，每组以 “.” 隔开，再将每组数转换为十进制数。

##### 4.3.2 IP 地址由网络和主机两部分标识组成

![image-20210701100353161](https://raw.githubusercontent.com/silence/blog/assets/assets/20210701100353.png)

##### 4.3.3 IP 地址的分类

- A 类 IP 地址是首位以 “0” 开头的地址，从第 1 位到第 8 位是它的网络标识，A 类地址的后 24 位相当于主机标识。一个网段内可容纳的主机地址上限为 16777214 个。0.0.0.0 ~ 127.0.0.0 是 A 类的网络地址。

- B 类 IP 地址是前两位为 “10” 的地址。从第 1 位到第 16 位是它的网络标识。B 类地址的后 16 位相当于主机标识。因此一个网段内可容纳的主机地址上限为 65534 个。128.0.0.1 ~ 191.255.0.0 是 B 类的网络地址。
- C 类 IP 地址是前三位为 “110” 的地址。从第 1 位到第 24 位是它的网络标识。C 类地址的后 8 位相当于主机标识。因此一个网段内可容纳的主机地址上限为 254 （去掉全为 0 和全为 1 的情况）个。192.168.0.0 ~ 239.255.255.0 是 C 类的网络地址。
- D 类 IP 地址是前四位为 “1110” 的地址。从第 1 位到第 32 位是它的网络标识。D 类没有主机标识，常用于多播。224.0.0.0 ~ 239.255.255.255 是 D 类的网络地址。

![image-20210701101656261](https://raw.githubusercontent.com/silence/blog/assets/assets/20210701101656.png)

##### 4.3.6 子网掩码

子网掩码用二进制方式表示，也是一个 32 位的数字。它对应 IP 地址网络标识部分的位全部为 “1”，对应 IP  地址主机标识部分全部为 “0”。

![image-20210702095522159](https://raw.githubusercontent.com/silence/blog/assets/assets/20210702095522.png)

##### 4.3.8 全局地址与私有地址

![image-20210702095812658](https://raw.githubusercontent.com/silence/blog/assets/assets/20210702095812.png)

包含在这个范围内的 IP 地址都属于私有 IP，而在此之外的 IP 地址称为全局 IP。

#### 4.4 路由控制

路由控制表的形成方式有两种：一种是管理员手动设置，另一种是路由器与其他路由器相互交换信息时自动刷新。前者加静态路由控制，后者叫动态路由控制。

![image-20210702100708144](https://raw.githubusercontent.com/silence/blog/assets/assets/20210702100708.png)

#### 4.5 IP 分割处理与再构成处理

MTU: 最大传输单元。

![image-20210702101520022](https://raw.githubusercontent.com/silence/blog/assets/assets/20210702101520.png)

##### 4.5.3 路径 MTU 发现

![image-20210702101858990](https://raw.githubusercontent.com/silence/blog/assets/assets/20210702101859.png)![image-20210702101914321](https://raw.githubusercontent.com/silence/blog/assets/assets/20210702101914.png)

#### 4.6 IPv6

IPv6 的 IP 地址长度为 128 位。它所能表示的数字高达 38 位数。

一个 IPv6 地址用 16 进制表示：

```
FEDC: BA98: 7654: 3210: FEDC: BA98: 7654: 3210
```

#### 4.7 IPv4 首部

![image-20210703124021092](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703124021.png)

#### 4.8 IPv6 首部

![image-20210703124647465](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703124647.png)

### 第 5 章 IP 协议相关技术

#### 5.2 DNS

![image-20210703134614359](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703134614.png)

##### 5.2.4 DNS 查询

![image-20210703135136354](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703135136.png)

#### 5.3 ARP

##### 5.3.1 ARP 概要

ARP (Address Resolution Protocol) 是一种解决地址问题的协议。以目标 IP 地址为线索，用来定位下一个应该接收数据分包的网络设备对应的 MAC 地址。

##### 5.3.2 ARP 的工作机制

![image-20210703135610188](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703135610.png)

#### 5.4 ICMP

![image-20210703140212759](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703140212.png)

#### 5.5 DHCP

自动设置 IP 地址、统一管理 IP 地址分配，就产生了 DHCP (Dynamic Host Configuration Protocol) 协议。

DHCP 在分配 IP 地址又两种方法，一种是由 DHCP 服务器在特定的 IP 地址中自动选出一个进行分配。另一种方法是针对 MAC 地址分配一个固定的 IP 地址。而且这两种方法可以并用。

#### 5.6 NAT

NAT (Network Address Translator) 是用于在本地网络中使用私有地址，在连接互联网时转而使用全局 IP 地址的技术。除转换 IP 地址外，还出现了可以转换 TCP、UDP 端口号的 NAPT (Network Address Ports Translator) 技术。由此可以实现一个全局 IP 地址与多个主机的通信。

![image-20210703141307031](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703141307.png)

### 第 6 章 TCP 与 UDP

TCP/IP 或 UDP/IP 通信中通常采用 5 个信息来识别一个通信。它们是“源 IP 地址”、“目标 IP 地址”、“协议号”、“源端口号”、“目标端口号”。只要其中某一项不同，则被认为是其他通信。

TCP 建立连接与断开

![image-20210703163801109](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703163801.png)

##### 6.4.5 TCP 以段为单位发送数据

MSS: Maximum Segment Size。最大消息长度。

![image-20210703164020974](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703164021.png)

##### 6.4.6 利用窗口控制提高速度

![image-20210703164150189](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703164150.png)

![image-20210703164859964](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703164900.png)

##### 6.4.7 窗口控制与重发控制

发送端的主机如果连续 3 次收到同一个确认应答，就会将其所对应的数据进行重发。

![image-20210703164505971](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703164506.png)

称为高速重发控制。

##### 6.4.8 流控制

![image-20210703165112000](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703165112.png)

##### 6.4.9 拥塞控制

慢启动

![image-20210703165430438](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703165430.png)

![image-20210703165614680](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703165614.png)

#### 6.6 UDP 首部的格式

![image-20210703170003985](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703170004.png)

#### 6.7 TCP 首部的格式

![image-20210703170042052](https://raw.githubusercontent.com/silence/blog/assets/assets/20210703170042.png)

### 第 7 章 路由协议

生成路由控制表的算法。

### 第 8 章 应用协议

