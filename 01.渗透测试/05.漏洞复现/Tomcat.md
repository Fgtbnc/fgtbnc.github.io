---
title: Tomcat漏洞
date: 2023/6/1 19:50:25
categories:
- 漏洞复现
tags:
- Tomcat
---



## 环境搭建

vulhub



## Tomat了解

### 目录结构

![image-20230228234754791](../../../images/image-20230228234754791-1686995572132.png)

webapps

![image-20230617184342638](../../../images/image-20230617184342638.png)



```
1. bin目录: 存放一些二进制的文件，例如Tomcat常用的 启动脚本: startup.bat或startup.sh 关闭脚本: shutdown.bat 或 shutdown.sh等等

2. conf目录: 存放的是Tomcat的配置文件

server.xml：可以设置端口号、设置域名或IP、默认加载的项目、请求编码
web.xml：部署描述文件，这个web.xml中描述了一些默认的servlet，部署每个webapp时，都会调用这个文件，配置该web应用的默认servlet
context.xml：可以用来配置数据源之类的
tomcat-users.xml：用来配置管理tomcat的用户与权限
Catalina:可以设置默认加载的项目


3. lib目录: 存放的是全局的jar包

4. logs目录: 存放的是Tomcat的日志，如果Tomcat出错什么的，就需要在这里的日志中查找问题

5. temp目录: 存放的是临时性的文件

6. webapps目录: 存放的是Java的Web项目，要部署的项目就需要放在这个目录当中


7. work目录: 存放的是由JSP代码翻译的Java代码，以及编译的.class文件
```





## fofa语法

```
title="Apache Tomcat"
server="Tomcat" 

版本号：在Tomcat后面添加/x.x
```





## Tomcat漏洞

### manager弱口令+部署war包getshell

#### 影响版本

Tomcat7+或者配置错误



#### 复现

> conf/tomcat-users.xml中保存了登录凭证，导致可能存在弱口令漏洞，从而能访问Manager APP上传后门。

![image-20230228232924268](../../../images/image-20230228232924268-1686995572133.png)

进入manager界面后可以部署war包

​	![image-20230228232939975](../../../images/image-20230228232939975-1686995572134.png)

> war包为jsp压缩文件（`jar cvf xxx.jsp`），Tomcat会自动解压，
>
> 如将shell.jsp打包为shell.war，Tomcat会将其解压为/shell/shell.jsp

![image-20230228233506559](../../../images/image-20230228233506559-1686995572134.png)



#### 修复方案

删除manage目录或者修改conf/tomcat-users.xml中的用户凭证



### PUT任意文件上传（CVE-2017-12615）

#### 影响版本

7.0.0 <= tomcat <=7.0.79或者配置错误

#### 复现

conf/web.xml文件配置了readonly=false，导致可以写文件

![image-20230625214424587](../../../images/image-20230625214424587.png)

```http
PUT /1.jsp/ HTTP/1.1
Host: 192.168.174.129:8080
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 693

shell
```

> **需要进行绕过**
>
> Windows：
> 1、利用/shell.jsp::$DATA的方式绕过
> 2、/shell.jsp%20，空格绕过
> 3、/shell.jsp/ ， Tomcat在处理文件时会删除最后的/
> Linux：
> 1、/shell.jsp/ ， Tomcat在处理文件时会删除最后的/

![img](../../../images/2676572-20220223152639690-1145929483.png)

返回201表示上传成功



#### 修复方案

conf/web.xml文件配置readonly值为True或注释该参数



### AJP文件包含/读取--Ghostcat幽灵猫（CVE-2020-1938）

#### 影响版本

![image-20230617182333230](../../../images/image-20230617182333230.png)

#### 漏洞危害

![image-20230913153055256](../../../images/image-20230913153055256.png)

#### 复现

**python2运行**

**可以读取或包含 Tomcat 上所有 webapp 目录下的任意文件**

![image-20230617184142564](../../../images/image-20230617184142564.png)

![image-20230617184129890](../../../images/image-20230617184129890.png)



#### 漏洞检测与修复

[CVE-2020-1938:Tomcat AJP协议文件包含漏洞分析 | 回忆飘如雪 (gv7.me)](https://gv7.me/articles/2020/cve-2020-1938-tomcat-ajp-lfi/#2-2-任意代码执行)