---
title: DNS协议
date: 2023/6/1 19:48:25
categories:
- 计算机基础
tags:
- DNS协议
---



# DNS协议介绍

DNS（Domain Name System）协议位于ISO参考模型的应用层。在ISO参考模型中，应用层是最高层，负责提供网络服务和应用程序之间的接口。

DNS协议的主要功能是将域名解析为IP地址。当用户在浏览器中输入一个域名时，例如"[example.com](http://example.com/)"，浏览器会发送一个DNS查询请求到本地DNS服务器。本地DNS服务器通过DNS协议向上级DNS服务器发出请求，逐级查询，直至找到与域名对应的IP地址，并返回给用户的设备。这样，用户可以通过域名来访问互联网上的资源，而不需要记住复杂的IP地址。

DNS协议支持UDP和TCP，端口号为53



# 域名分类

![image-20221115163957139](../images/image-20221115163957139-1694328360129.png)



# DNS TTL

TTL值全称是“生存时间（Time To Live)”，简单的说它表示DNS记录在DNS服务器上缓存时间，数值越小，修改记录各地生效时间越快。
当各地的DNS(LDNS)服务器接受到解析请求时，就会向域名指定的授权DNS服务器发出解析请求从而获得解析记录；该解析记录会在DNS(LDNS)服务器中保存一段时间，这段时间内如果再接到这个域名的解析请求，DNS服务器将不再向授权DNS服务器发出请求，而是直接返回刚才获得的记录；而这个记录在DNS服务器上保留的时间，就是TTL值。

常见的设置TTL值的场景：
• 增大TTL值，以节约域名解析时间
• 减小TTL值，减少更新域名记录时的不可访问时间



# 浏览器DNS解析流程

因为每次请求时都向DNS服务器发起查询，过于浪费资源，所以出现了DNS缓存。

**浏览器DNS查找顺序一般是这样的:** 

   浏览器DNS缓存->本地系统DNS缓存->本地计算机HOSTS文件->路由器DNS缓存->ISP的DNS缓存->根服务器递归搜索

- 浏览器DNS缓存（内存中)

  浏览器会按照一定频率缓存DNS记录  [查看google浏览器DNS缓存](https://www.jianshu.com/p/9e7aa4ec4b46)

- 本地DNS缓存(内存中)

  查看：`ipconfig /displaydns`

  清除：`ipconfig /flushdns`

- 本地HOSTS文件

  `c:\windows\system32\drivers\etc\hosts ` 

- 路由器DNS缓存

  > 路由器DNS被篡改会造成域名劫持，你访问的网址都会被定位到同一个位置，但是IP直接可以访问

- ISP的DNS服务器

  - 公共服务器

    `8.8.8.8` (Google 提供)

    `114.114.114.114` (国内公共 DNS)

  - 专用服务器

- 根服务器

  > 以访问www.baidu.com为例， DNS服务器先问根域名服务器.com域名服务器的IP地址，然后再问.com域名服务器，以此类推
  >



![image-20221115164624383](../images/image-20221115164624383-1694328362681.png)



# 设置DNS解析

以阿里云服务器为例

![image-20221004160123895](../images/image-20221004160123895-1694328366241.png)

- 主机记录

  就是要解析的域名，比如网站根域名为khaz.top，主机记录@即@khaz.top等价于khaz.top

  > 通常会将加www和不加www的域名都解析到同一个ip上。

- 记录类型

  ![image-20221004160123896](../images/image-20221004160443733-1694328407151.png)

- 解析线路

  就是选择哪一个ISP的DNS服务器。

- 记录值

  就是域名对应的IP地址。

- TTL

  DNS缓存时间，当修改DNS解析时，需要经过TTL时间才会生效。

  

# 问题

#### 当多个域名解析到一个IP时，如果通过IP地址来访问服务器，那么会访问到哪一个域名（网站）呢？

答案：

1. 虚拟主机技术

   实现多个站点在同一台服务器上。

   > 比如服务器使用Apache，那么在Apache的配置文件中加入VirtualHost即可新增虚拟主机

   ```php
   <VirtualHost *:80>
    DocumentRoot /var/www/acm
    ServerName acm.hdu.edu.cn
   </VirtualHost>
       
   <VirtualHost *:80>
    DocumentRoot /var/www/html
    ServerName www.hdu.edu.cn
   </VirtualHost>
   ```

   ServerName对应请求头的HOST字段，DocumentRoot对应的站点目录

   这样HOST为acm.hdu.edu.cn时，访问的就是/var/www/acm下的acm.hdu.edu.cn网站

   HOST为www.hdu.edu.cn时，访问的就是/var/www/html下的www.hdu.edu.cn网站

   > 虚拟主机技术，也可以实现不同端口对应不同站点，只要修改上面的<VirtualHost *:80>即可。

2. 反向代理技术

   每个站点都在不同的主机上，但都是通过E这个代理服务器进行访问的。

   ![反向代理流](../images/reverse-proxy-flow-1694328428983.svg)



# 安全相关

## DNS欺骗攻击 

DNS欺骗攻击（DNS spoofing attack）是一种网络攻击方式，攻击者通过篡改或伪造域名系统（DNS）的解析结果，将用户的请求重定向到恶意网站或进行信息窃取等恶意行为。这种攻击方式可能导致用户被引导到虚假的网站，从而遭受钓鱼诈骗、恶意软件下载、个人信息泄露等风险。



## DNS  Rebinding

**有漏洞的SSRF过滤器执行步骤如下**

1. 获取输入的URL，从该URL中提取HOST，如果提取出来的是IP，那么直接跳到第三步；
2. 对该HOST进行DNS解析，获取到解析的IP；
3. 检测该IP是否是合法的，比如是否是私有IP等（是就直接终止流程）；
4. 如果IP检测为合法的，则进入CURL发包；

**漏洞点**
DNS解析一共分两次，其中第一次是至关重要的有效性检测，第二次则是具体发起的请求。我们利用DNS Rebinding技术，在**第一次校验IP**的时候**返回一个合法的IP**，在**真实发起请求**的时候，返回我们**真正想要访问的内网IP**即可。

**实现方式**

- 同一个域名绑定两条TTL都是0的A记录，不过这样DNS解析是随机的，不够稳定
- 通过自建DNS服务器，稳定控制解析返回结果



## DNS Log

ns记录指向自建的DNS服务器，然后当目标主机发起DNS Query时，自建的DNS服务器就记录下该请求。从下图中可以看到向NS服务器发起DNS请求的为用户设置的DNS服务器，这也是为什么查看DNSLog记录的IP不是目标主机的原因。

![20210615113242-57d33758-cd8a-1](../images/20210615113242-57d33758-cd8a-1.png)

# 常用命令

nslookup/dig

https://blog.csdn.net/weixin_42426841/article/details/115364502















