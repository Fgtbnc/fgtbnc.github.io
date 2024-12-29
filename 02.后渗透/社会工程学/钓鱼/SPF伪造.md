---
title: 初探邮件钓鱼
date: 2023/11/10 10:26
categories:
- 渗透测试
tags:
- 钓鱼
---

# SMTP协议

[邮件实现详解（一）------邮件发送的基本过程与概念 - YSOcean - 博客园](https://www.cnblogs.com/ysocean/p/7652934.html)

[邮件实现详解（二）------手工体验smtp和pop3协议 - YSOcean - 博客园](https://www.cnblogs.com/ysocean/p/7653252.html)



SMTP协议使用的端口

- **端口 25** 是 SMTP 服务器之间连接最常用的端口。如今，终端用户网络的防火墙通常会阻止这个端口，因为垃圾邮件发送者试图滥用它来发送大量垃圾邮件。
- **端口 465** 曾经专门用于具备[安全套接字层（SSL）](https://www.cloudflare.com/learning/ssl/what-is-ssl/)加密的 SMTP。但 SSL 已被[Transport Layer Security（TLS）](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/)取代，因而现代电子邮件系统不再使用这个端口。该端口仅出现在传统（过时）系统中。
- **端口 587** 现在是电子邮件提交的默认端口。经过该端口的 SMTP 通信使用 TLS 加密。
- **端口 2525** 与 SMTP 没有正式关联，但某些电子邮件服务在上述端口被堵塞的情况下，通过这个端口提供 SMTP 传输。



# SPF协议

## 由来与原理

spf全称为（Sender Policy Framework），即发件人策略框架

目前所进行的邮件通信，使用的是smtp协议（Simple Mail Transfer Protocol），即简单邮件传输协议。但是smtp是没有很好的安全措施的，一个简单的例子为：发件人的邮箱地址可以由发信方任意声明，即发件人邮箱伪造。

spf就是为了解决这个问题，spf的原理：

假设邮件服务器收到了一封邮件，发件人的IP为：192.6.6.6，并且声称发件人为 [email@example.com](mailto:email@example.com)。为了确认发件人不是伪造的，邮件服务器会查询 `example.com`的spf记录。如果该域的spf记录设置允许IP为 192.6.6.6 主机发送邮件，则服务器认为这封邮件是合法的，否则，会退信（即收件人收不到邮件），或者邮件躺在垃圾箱。邮箱伪造可以声明他来自`example.com`，但是却无法操作 `example.com` 的 DNS解析记录，也无法伪造自己的IP地址，所以SPF还是可以有效防御邮件伪造的。

通俗的来讲，A 和 B 两个人建立通信，A向B发送信息，B 检查 A是不是在常用地(白名单)发的信息，如果不是，则拒收信息。

![Sender-policy-framework-spf-exchange-2](../images/Sender-policy-framework-spf-exchange-2-1699623811864.jpg)

## 语法

一条 SPF 记录定义了一个或者多个 mechanism，而 mechanism 则定义了哪些 IP 是允许的，哪些 IP 是拒绝的。

这些 mechanism 包括以下几类：

```
all | ip4 | ip6 | a | mx | ptr | exists | include
```

每个 mechanism 可以有四种前缀：

```
"+"  Pass（通过）
"-"  Fail（拒绝）
"~"  Soft Fail（软拒绝）
"?"  Neutral（中立）
```

测试时，将从前往后依次测试每个 mechanism。如果一个 mechanism 包含了要查询的 IP 地址（称为命中），则测试结果由相应 mechanism 的前缀决定。默认的前缀为`+`。如果测试完所有的 mechanisms 也没有命中，则结果为 Neutral。

除了以上四种情况，还有 None（无结果）、PermError（永久错误）和 TempError（临时错误）三种其他情况。对于这些情况的解释和服务器通常的处理办法如下：

| 结果      | 含义                                     | 服务器处理办法       |
| --------- | ---------------------------------------- | -------------------- |
| Pass      | 发件 IP 是合法的                         | 接受来信             |
| Fail      | 发件 IP 是非法的                         | 退信                 |
| Soft Fail | 发件 IP 非法，但是不采取强硬措施         | 接受来信，但是做标记 |
| Neutral   | SPF 记录中没有关于发件 IP 是否合法的信息 | 接受来信             |
| None      | 服务器没有设定 SPF 记录                  | 接受来信             |
| PermError | 发生了严重错误（例如 SPF 记录语法错误）  | 没有规定             |
| TempError | 发生了临时错误（例如 DNS 查询失败）      | 接受或拒绝           |

注意，上面所说的「服务器处理办法」仅仅是 SPF 标准做出的建议，并非所有的邮件服务器都严格遵循这套规定。

### Mechanisms

下面介绍上面提到的 mechanism：

#### all

表示所有 IP，肯定会命中。因此通常把它放在 SPF 记录的结尾，表示处理剩下的所有情况。例如：

```
"v=spf1 -all" 拒绝所有（表示这个域名不会发出邮件）
"v=spf1 +all" 接受所有（域名所有者认为 SPF 是没有用的，或者根本不在乎它）
```

#### ip4

格式为`ip4:<ip4-address>`或者`ip4:<ip4-network>/<prefix-length>`，指定一个 IPv4 地址或者地址段。如果`prefix-length`没有给出，则默认为`/32`。例如：

```
"v=spf1 ip4:192.168.0.1/16 -all"
只允许在 192.168.0.1 ~ 192.168.255.255 范围内的 IP
```

#### ip6

格式和`ip4`的很类似，默认的`prefix-length`是`/128`。例如：

```
"v=spf1 ip6:1080::8:800:200C:417A/96 -all"
只允许在 1080::8:800:0000:0000 ~ 1080::8:800:FFFF:FFFF 范围内的 IP
```

#### a 和 mx

这俩的格式是相同的，以`a`为例，格式为以下四种之一：

```
a
a/<prefix-length>
a:<domain>
a:<domain>/<prefix-length>
```

会命中相应域名的 a 记录（或 mx 记录）中包含的 IP 地址（或地址段）。如果没有提供域名，则使用当前域名。例如：

```
"v=spf1 mx -all"
允许当前域名的 mx 记录对应的 IP 地址。

"v=spf1 mx mx:deferrals.example.com -all"
允许当前域名和 deferrals.example.com 的 mx 记录对应的 IP 地址。

"v=spf1 a/24 -all"
类似地，这个用法则允许一个地址段。
```

例如，这是一个比较常见的 SPF 记录，它表示支持当前域名的 a 记录和 mx 记录，同时支持一个给定的 IP 地址；其他地址则拒绝：

```
v=spf1 a mx ip4:173.194.72.103 -all
```

#### include

格式为`include:<domain>`，表示引入`<domain>`域名下的 SPF 记录。注意，如果该域名下不存在 SPF 记录，则会导致一个`PermError`结果。例如：

```
"v=spf1 include:example.com -all" 即采用和 example.com 完全一样的 SPF 记录
```

#### exists

格式为`exists:<domain>`。将对`<domain>`执行一个 A 查询，如果有返回结果（无论结果是什么），都会看作命中。

#### ptr

格式为`ptr`或者`ptr:<domain>`。使用`ptr`机制会带来大量很大开销的 DNS 查询，所以连官方都不推荐使用它。

### 关于v=spf1

这是必须的，这个表示采用 SPF 1 版本，现在它的最新版本就是第 1 版。

### Modifiers

SPF 记录中还可以包括两种可选的 modifier；一个 modifier 只能出现一次。

#### redirect

格式为`redirect=<domain>`

将用给定域名的 SPF 记录替换当前记录。

#### exp

格式为`exp=<domain>`，目的是如果邮件被拒绝，可以给出一个消息。而消息的具体内容会首先对`<domain>`执行 TXT 查询，然后执行宏扩展得到。



### Demo

查询对应域名是否有SPF记录

```bash
nslookup -type=txt domain
```

![image-20231110214612932](../images/image-20231110214612932.png)

> 找到v=spf1（表示采用 SPF 1 版本）的txt记录

SPF配置如下

```bash
v=spf1 ip4:45.249.212.32 ip4:45.249.212.35 ip4:45.249.212.255 ip4:45.249.212.187/29 ip4:45.249.212.191 ip4:185.176.76.210 ip4:168.195.93.47 ip4:103.69.140.247 -all
```

解释：依次匹配ipv4地址，如果ip地址全不匹配就拒绝





## 存在风险的SPF记录

### 未设置SPF

![image-20231110213515820](../images/image-20231110213515820.png)

可以直接伪造邮件

### 设置不当--使用了不恰当的限定词`~`,`+`

From https://saucer-man.com/information_security/452.html#cl-6

![image-20231110174320801](../images/image-20231110174320801.png)

![image-20231110174408422](../images/image-20231110174408422.png)

### SPF绕过进阶

https://www.freebuf.com/articles/system/238215.html



# 实践

## swaks使用

```bash
--from test@qq.com 				//发件人邮箱；

--ehlo qq.com 					//伪造邮件ehlo头，填发件人邮箱的域名

--body "http://www.baidu.com" 	//引号中的内容即为邮件正文

--header "Subject:hello" 		//邮件头信息，subject为邮件标题

--data test.eml      //将正常源邮件的内容保存作为正常邮件发送

--server mail.smtp2go.com -p 2525 -au 用户名  -ap 密码  //邮箱服务器代发，用于绕过SPF
```



## 基本流程

自己用邮箱编辑好钓鱼邮件，然后先发送给自己，然后查看已发送邮件，点击导出为eml文件：

![image-20231110215714713](../images/image-20231110215714713.png)

编辑eml文件，删除from值之前的信息，然后修改From和To的内容（这两是看到的收件人和发件人）

![image-20231110213433252](../images/image-20231110213433252.png)

使用如下命令发送

无SPF

```bash
swaks --to xxx@qq.com --from jiaowuchu@swust.edu.cn --data aaa.eml --ehlo jiaowuchu@swust.edu.cn
```

效果

![image-20231110212839506](../images/image-20231110212839506.png)

邮件原文

![image-20231110213412228](../images/image-20231110213412228.png)



有SPF

```bash
# kali中自带
swaks --to xxx@qq.com --from hr@huawei.com --data aaa.eml --server mail.smtp2go.com -p 2525 -au <username> -ap <password>
```





# 总结

这里学习了一下如何进行邮件伪造

还需要学习的一些问题

- 如何找到目标人员的邮箱地址

  https://forum.butian.net/share/351

- 如何增加邮件可信度，附件木马的制作

  https://mp.weixin.qq.com/s/hi1YgUUHnFDGf26cUXJkQQ

- 附件木马的免杀



# 参考文章

https://saucer-man.com/information_security/452.html#cl-8

https://service.mail.qq.com/detail/124/995

http://www.renfei.org/blog/introduction-to-spf.html