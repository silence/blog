本文为 《图解 HTTP》的重点摘抄

[TOC]

### 第 1 章 了解 Web 及网络基础

#### 1.1 使用 HTTP 协议访问 Web

HTTP (HyperText Transfer Protocol，超文本传输协议)

#### 1.2 HTTP 的诞生

##### 1.2.1 为知识共享而规划 Web

1989 年 3 月，CER（欧洲核子研究组织）的 （Tim BernersLee）博士提出了一种能让远隔两地的研究者们共享知识的设想。

WWW (World Wide Web，万维网)

HTML (HyperText Markup Language，超文本标记语言)

URL (Uniform Resource Locator，统一资源定位符)

#### 1.3 网络基础 TCP/IP

##### 1.3.1 TCP/IP 协议族

计算机网络设备要相互通信，双方就必须基于相同的方法。我们把这种通信的规则称为 **协议** (protocol)。

##### 1.3.2 TCP/IP 的分层管理

TCP/IP 协议族按层次分别分为以下 4 层：**应用层、传输层、网络层和数据链路层。**

**应用层决定了向用户提供应用服务时通信的活动。**

FTP (File Transfer Protocol，文件传输协议)、DNS (Domain Name System，域名系统) 就是应用层中的两类。

**传输层对上层应用层，提供处于网络连接中的两台计算机之间的数据传输。**

在传输层中有两个性质不同的协议：TCP (Transmission Control Protocol，传输控制协议) 和 UDP (User Data Protocol，用户数据报协议)。

**网络层用来处理在网络上流动的数据包。**

数据包是网络传输的最小数据单位。该层规定了通过怎样的路径（所谓的传输路线）到达对方计算机，并把数据包传送给对方。

**链路层用来处理连接网络的硬件部分。**

NIC (Network Interface Card，网络适配器，即网卡)

硬件上的范畴均在链路层的作用范围之内。

##### 1.3.3 TCP/IP 通信传输流

![image-20210617235316789](https://raw.githubusercontent.com/silence/blog/assets/assets/20210617235316.png)

![image-20210617235412327](https://raw.githubusercontent.com/silence/blog/assets/assets/20210617235412.png)

这种把数据包装起来的做法称为封装（encapsulate）。

#### 1.4 与 HTTP 关系密切的协议：IP、TCP 和 DNS

##### 1.4.1 负责传输的 IP 协议

IP (Internel Protocol，网际协议)

MAC 地址 (Media Access Control Address)

IP 地址指明了节点被分配到的地址，MAC 地址是指网卡所属的固定地址。

使用 ARP 协议凭借 MAC 地址进行通信。

在网络上，通信的双方在同一局域网（LAN）内的情况是很少的，通常是经过多台计算机和网络设备中转才能连接到对方。而在进行中转时，会利用下一站中转设备的 MAC 地址来搜索下一个中转目标。这时，会采用 ARP 协议（Address Resolution Protocol）。ARP 是一种用以解析地址的协议，根据通信方的 IP 地址就可以反查出对应的 MAC 地址。

![image-20210618001654249](https://raw.githubusercontent.com/silence/blog/assets/assets/20210618001654.png)

在到达通信目标前的中转过程，有点像快递公司的送货过程，寄快递的人只需要将货物送到快递公司，快递公司再将货物送到下一个集散中心，最后送达对方手中。这种机制称为 路由选择（routing）。

无论哪台计算机、哪台网络设备，它们都无法全面掌握互联网中的细节。

##### 1.4.2 确保可靠性的 TCP 协议

按层次分，TCP 位于传输层，提供可靠的字节流服务。

所谓的字节流（Byte Stream Service）是指，为了方便传输，将大块数据分割成以报文段（segment）为单位的数据包进行管理。

为了确保能到达目标

TCP 协议采用了三次握手（three-way handshaking）策略。握手过程中使用了 TCP 的标志（flag）--- SYN (synchronize) 和 ACK (acknowledgement)。

![image-20210618003343230](https://raw.githubusercontent.com/silence/blog/assets/assets/20210618003343.png)

除了上述三次握手，TCP 协议还有其他各种手段来保证通信的可靠性。

#### 1.5 负责域名解析的 DNS 服务

![image-20210618134210110](https://raw.githubusercontent.com/silence/blog/assets/assets/20210618134210.png)

#### 1.6 各种协议与 HTTP 协议的关系

![image-20210618134302007](https://raw.githubusercontent.com/silence/blog/assets/assets/20210618134302.png)

#### 1.7 URI 与 URL

URI 是 Uniform Resource Identifier （统一资源标识符） 的缩写。RFC2396 分别对这 3 个单词进行了定义。

URI 用字符串标识某一互联网资源，而 URL 表示资源的地点（互联网上所处的位置）。可见 URL 是 URI 的子集。

几种 URI 例子如下所示：

```
ftp://ftp.is.co.za/rfc/rfc1808.txt
http://www.ietf.org/rfc/rfc2396.txt
ldap://[2001:db8::7]/c=GB?objectClass?one
mailto:John.Doe@example.com
news:comp.infosystems.www.servers.unix
tel:+1-816-555-1212
telnet://192.0.2.16:80/
urn:oasis:names:specification:docbook:dtd:xml:4.1.2
```

我们先来了解一下绝对 URI 的格式。

![image-20210618234022917](https://raw.githubusercontent.com/silence/blog/assets/assets/20210618234023.png)

**RFC** (Request for Comments，征求修正意见书)

### 第 2 章 简单的 HTTP 协议

请求报文的构成

![image-20210619125408583](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619125408.png)

响应报文的构成

![image-20210619151451013](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619151451.png)

#### 2.3 HTTP 是不保存状态的协议

stateless (无状态协议)

使用 HTTP 协议，每当有新的请求发送过来时，就会有对应的新响应产生。协议本身并不保留之前一切的请求或响应报文的信息。这是为了更快地处理大量事务，确保协议的可伸缩性。

OPTIONS 询问支持的方法

OPTIONS 方法用来查询针对请求 URI 指定的资源支持的方法。

##### 2.7.1 持久连接

![image-20210619153404242](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619153404.png)

##### 2.7.2 管线化

![image-20210619153504962](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619153504.png)

### 第 3 章 HTTP 报文内的 HTTP 信息

空行 CR + LF （Carriage Return，回车符，Line Feed，换行符）

#### 3.5 获取部分内容的范围请求

Range 字段

![image-20210619154151604](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619154151.png)

### 第 4 章 HTTP 状态码

- 200 OK

- 204 No Content

  代表服务器接收的请求已成功处理，但在返回的响应报文中不含实体的主体部分。

  **一般在只需要客户端往服务器发送信息，而对客户端不需要发送新信息内容的情况下使用**。

- 206 Partial Content

  ![image-20210619154547901](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619154547.png)

- 301 Moved Permanently

  永久性重定向

- 302 Found

  临时重定向

- 303 See Other

  ![image-20210619154822400](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619154822.png)

  比如，当使用 POST 方法访问 CGI 程序，其执行后的处理结果是希望客户端能以 GET 方法重定向到另一个 URL 上去时，返回 303 状态码。

- 304 Not Modified

  304 状态码返回时，不包含任何响应的主体。

- 307 Temporary Redirect

  临时重定向。该状态码与 302 Found 有着相同的含义。302 标准禁止 POST 变换成 GET，(尽管大家不遵守)。307 会遵照浏览器标准，不会从 POST 变成 GET。

- 400 Bad Request

  表示请求报文中存在语法错误。

- 401 Unauthorized

  表示发送的请求需要有通过 HTTP 认证（BASIC 认证、DIGEST 认证）的认证信息。

- 403 Forbidden

  ![image-20210619155735321](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619155735.png)

- 404 Not Found

  ![image-20210619155826277](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619155826.png)

- 500 Internal Server Error

  ![image-20210619155853869](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619155853.png)

- 503 Service Unavailable

  ![image-20210619155930624](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619155930.png)

  该状态码表明服务器暂时处于超负载或正在进行停机维护。

### 第 5 章 与 HTTP 协作的 Web 服务器

- 代理

![image-20210619160854044](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619160854.png)

- 网关

![image-20210619160939505](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619160939.png)

- 隧道

![image-20210619161016449](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619161016.png)

### 第 6 章 HTTP 首部

#### 6.4 请求首部字段

- Cache-Control 指令一览

  ![image-20210619161815831](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619161815.png)

  ![image-20210619161831796](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619161831.png)

- 附带条件请求

  形如 If-xxx 这种样式的请求首部字段，都可称为条件请求。

  ![image-20210619162001750](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619162001.png)

  - If-Match

    ![image-20210619162050497](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619162050.png)

    **只有当 If-Match 的字段值跟 ETag 值匹配一致时，服务器才会接受请求。**

  - If-Modified-Since

    ![image-20210619162231399](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619162231.png)

    **如果在 If-Modified-Since 字段指定的日期时间后，资源发生了更新，服务器会接受请求，否则返回 304 Not Modified 的响应。**

  - If-None-Match

    ![image-20210619162504421](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619162504.png)

    只有在 If-None-Match 的字段值与 ETag 值不一致时，可处理该请求。**与 If-Match 首部字段的作用相反。**

  - If-Range

    ![image-20210619162639676](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619162639.png)

    ![image-20210619162801952](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619162802.png)

  - If-Unmodified-Since

    与 If-Modified-Since 作用相反。

#### 6.5 响应首部字段

- Accept-Ranges

  ![image-20210619163307756](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619163307.png)

  当不能处理范围请求时，Accept-Ranges: none

- Age

  ![image-20210619163416901](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619163416.png)

- ETag

  ![image-20210619163518190](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619163518.png)

  Etag 中有强 ETag 值和 弱 ETag 值之分。

  强 ETag 值，不论实体发生多么细微的变化都会改变其值。

  ```
  ETag: "usagi-1234"
  ```

  弱 ETag 值只用于提示资源是否相同。只有资源发生了根本改变，产生差异时才会改变 ETag 值。这时，会在字段值最开始处附加 W/。

  ```
  ETag: W/"usagi-1234"
  ```

#### 6.7 为 Cookie 服务的首部字段

Set-Cookie 字段的属性

![image-20210619164654378](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619164654.png)

### 第 7 章 确保 Web 安全的 HTTPS

HTTP 主要有这些不足，例举如下。

- 通信使用明文（不加密），内容可能会被窃听。
- 不验证通信方的身份，因此有可能遭遇伪装。
- 无法证明报文的完整性，所以有可能已遭篡改。

SSL (Secure Socket Layer，安全套接层)

TLS (Transport Layer Security，安全层传输协议)

中间人攻击

![image-20210619165212204](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619165212.png)

#### 7.2 HTTP + 加密 + 认证 + 完整性保护 = HTTPS

使用两把密钥的公开密钥加密

![image-20210619165504689](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619165504.png)

HTTPS 采用混合加密机制

![image-20210619165611798](https://raw.githubusercontent.com/silence/blog/assets/assets/20210619165611.png)

##### 7.2.4 证明公开密钥正确性的证书

公开密钥加密方式还是存在一些问题的。那就是无法证明公开密钥本身就是货真价实的公开密钥。或许在公开密钥传输途中，真正的公开密钥已经被攻击者替换掉了。

为了解决上述问题，可以使用由数字证书认证机构（CA，Certificate Authority）和其相关机关颁发的公开密钥证书。

![image-20210620171751630](https://raw.githubusercontent.com/silence/blog/assets/assets/20210620171751.png)

##### 7.2.5 HTTPS 的安全通信机制

![image-20210620172254859](https://raw.githubusercontent.com/silence/blog/assets/assets/20210620172254.png)

下面是对整个流程的图解。图中说明了从仅使用服务器端的公开密钥证书（服务器证书）建立 HTTPS 通信的整个过程。

![image-20210620172535603](https://raw.githubusercontent.com/silence/blog/assets/assets/20210620172535.png)









