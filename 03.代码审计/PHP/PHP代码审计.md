---
title: PHP代码审计
date: 2023/5/31 19:48:25
categories:
- PHP
tags:
- 代码审计
---

- [x] BlueCMS

- [x] SeaCMS

- [x] DedeCMS

- [x] ThinkPHP

  

# 调试

https://blog.csdn.net/Xxy605/article/details/120973447



# 思路

From https://www.anquanke.com/post/id/265092#h3-5

- 函数集文件，通常命名包含function或者common等关键字，这些文件里面是一些公共的函数，提供其他文件统一调用，所以大多数文件都会在文件头部包含到其他文件。寻找这些文件一个非常好用的技巧就是去打开index.php或者一些功能性文件，在头部一般都能找到。
- 配置文件，通常命名中包括config关键字，配置文件包括web程序运行必须的功能性配置选项以及数据库等配置信息。从这个文件中可以了解程序的小部分功能，另外看这个文件的时候注意观察配置文件中参数值是单引号还是用双引号括起来，如果是双引号可能就存在代码执行的问题了。
- 安全过滤文件，安全过滤文件对代码审计至关重要，这关系到我们挖掘到的可以点能否直接利用，通常命名中带有filter、safe、check等关键字，这类文件主要是对参数进行过滤，大多数的应用其实会在参数的输入做一下addslashes()函数的过滤。
- index文件，index是一个程序的入口，所以通常我们只要读一读index文件就可以大致了解整个程序的架构、运行的流程、包含到的文件，其中核心的文件有哪些。而不同目录的index文件也有不同的实现方式，建议最好将几个核心目录的index文件都通读一遍。



# 总结大致流程

**1、先全局总览：入口文件、路由、全局处理方式等**
**2、定向功能审计：黑盒(找到敏感功能)+白盒（定位到代码进行审计）**
**3、敏感函数回溯**

先黑盒+白盒看敏感功能，再用自动化审计工具跑一遍并验证，最后再根据漏洞危险函数去回溯



### 自动化审计工具

RIPShttps://github.com/J0o1ey/rips-Chinese

seay

Fortify



# bluecms

## 黑盒测试

#### 用户注册

在注册时，邮箱插入xss代码

![image-20221202120912126](../../../images/image-20221202120912126-1686141975680.png)

![image-20221202120807928](../../../images/image-20221202120807928-1686141975682.png)

![image-20221202120844185](../../../images/image-20221202120844185-1686141975683.png)



#### 用户头像

上传文件，只允许上传图片后缀的文件，上传后有给出图片路径，那么不能配合.htaccess，需要找到一个文件包含的点来包含文件。

![image-20221202120844185](../../../images/image-20221202121559453.png)





#### 修改密码

是否存在越权,测试了没有



#### 发布新闻

![image-20221202120844185](../../../images/image-20221202122851275.png)

这里分类选择不了，应该是要管理员发布。





#### 目录扫描

管理员

![image-20221202123640697](../../../images/image-20221202123640697-1686141975683.png)

其他

![image-20221202125147298](../../../images/image-20221202125147298-1686141975683.png)







#### 后台登录

![image-20221202123831559](../../../images/image-20221202123831559-1686141975683.png)



这里没有验证码验证，那么可以很轻松地进行爆破，得到usernmae=admin，passwd=admin

![image-20221202120844185](../../../images/image-20221202124055555.png)



进去之后，可以看到一个建议，install文件夹，上面通过dirsearch也扫描出了这个文件，如果我们可以使用这个文件夹来重新安装网站，那么我们就可以直接重置管理员了，可惜是不可以的。

![image-20221202124235008](../../../images/image-20221202124235008-1686141975683.png)

#### 常用操作

发布本地新闻这里存在一个文件上传操作，但是也只能上传图片

![image-20221202175209000](../../../images/image-20221202175209000-1686141975683.png)

#### 会员管理

可以发现触发了之前的xss，那么如果我们没有登陆上后台，也可以通过这个xss来🎣获取管理员的cookie。

![image-20221202124632258](../../../images/image-20221202124632258-1686141975683.png)

也可以修改管理员密码，而且没有任何验证，可以直接修改。。

![image-20221202175052927](../../../images/image-20221202175052927-1686141975686.png)





## 白盒审计

先看一下目录

![image-20221202155440394](../../../images/image-20221202155440394-1686141975685.png)

### 前台

先看一下前台的index.php，内容差不多就是渲染页面，没有用户可控的参数，所以关注其包含的其他文件。

![image-20221202135553953](../../../images/image-20221202135553953-1686141975686.png)



跟进common.inc.php看一下

关键部分，用来验证用户身份，其中cookie中的参数是可控的。

![image-20221202135942047](../../../images/image-20221202135942047-1686141975683.png)

先判断cookie中是否有user_id，有的话再继续判断user_name和user_pwd是否存在，如果存在就check_cookie，如果只有user_name存在就查询名字是否只有一个，其他情况set_cookie。

#### sql注入

跟进check_cookie

![image-20221202140225964](../../../images/image-20221202140225964-1686141975683.png)

查username对应的密码，经过散列处理判断密码是否正确。



跟进getone--在include/mysql.class.php中

![image-20221202140336939](../../../images/image-20221202140336939-1686141975683.png)

![image-20221202140349680](../../../images/image-20221202140349680-1686141975683.png)

![image-20221202185616902](../../../images/image-20221202185616902-1686141975684.png)

可以看到没有对sql语句做任何处理，直接用mysql_query进行查询，sql语句错误时，返回错误消息Query error，然后是前面加了一个@，所以不能用报错注入。





再观察include/mysql.class.php可以发现

![image-20221202120844185](../../../images/image-20221202190550614.png)发现存在宽字节注入，那么这个CMS就可以随便注入了。。。（因为所有的sql操作都是用这个mysql类）



以ad_js.php为例

![image-20221202120844185](../imaages/image-20221202190230473.png)

![image-20221202190707634](../../../images/image-20221202190707634-1686141975684.png)





#### 文件包含+文件上传

虽然有白名单限制，只能上传图片，但是没有对文件内容进行过滤

![image-20221202171406255](../../../images/image-20221202171406255-1686141975684.png)

并且在user.php中，没有对$_POST['pay']进行过滤，存在目录穿越，就可以包含上传的用户头像。

![image-20221202185910243](../../../images/image-20221202185910243-1686141975684.png)



#### 任意文件删除

publish.php

![image-20221202190938927](../../../images/image-20221202190938927-1686141975684.png)

user.php

​	![image-20221202194855624](../../../images/image-20221202194855624-1686141975684.png)

需要先插入一条新闻，新闻名为要删除的文件的路径。





### 后台--/admin

还是先看一下index.php，也是只是输出了CMS和系统的一些信息，所以关注其包含的文件

![image-20221202191939851](../../../images/image-20221202191939851-1686141975684.png)

关键内容

![image-20221202192232288](../../../images/image-20221202192232288-1686141975684.png)

对输入的参数都使用addslashes()进行转义,不过还有`$SERVER`没有进行处理。

![image-20221202120844185](../../../images/image-20221202192459456.png)

​	使用session来验证admin身份。还是一样的使用check_cookie来判断，

​	![image-20221202120844185](../../../images/image-20221202140225964.png)

​	需要得到admin的pwd和cookie_hash，这两个都在数据库中，可以利用前台的sql注入获得，所以可以通过这个来登录admin用户。

#### 任意文件读取

![image-20221202195311948](../../../images/image-20221202195311948-1686141975684.png)

![image-20221202195409112](../../../images/image-20221202195409112-1686141975684.png)



#### 任意文件上传

![image-20221202120844185](../../../images/image-20221202191243192.png)

![image-20221202120844185](../../../images/image-20221202191403061.png)

文件路径只作了去除头尾空格的处理，存在目录穿越；文件内容实际上是调用了stripslashes，并没有过滤。

测试

![image-20221202120844185](../../../images/image-20221202194248639.png)

<img src="E:\typora img\image-20221202194300861.png" alt="image-20221202194300861" style="zoom:80%;" />





# SeaCMS

网站架构，无框架，目录结构如下

```php
├─admin   # 后台
├─css	  # 存放css文件
├─files	  # 存放页面
├─images  # 存放图片
├─inc	  # 存放网站配置，校验，过滤文件
├─install # 网站安装
├─seacmseditor # 网站编辑器
├─template # 网站模板
└─upload   # 存放上传文件
```

## 查看入口文件

所有的入口都是通过传递r参数来分发路由，对应files目录中的文件

![image-20230525134023397](../../../images/image-20230525134023397-1686141975684.png)

![image-20230525141822013](../../../images/image-20230525141822013-1686141975684.png)



这里只用`addslashes`进行了转义，明显路径穿越+文件包含

![image-20230525134741282](../../../images/image-20230525134741282-1686141975685.png)

其他思路

- 如果是linux系统，也可以通过`pearcmd.php`来getshell

  > 在7.3及以前，pecl/pear是默认安装的；在7.4及以后，需要我们在编译PHP的时候指定--with-pear才会安装

- 尝试%00截断路径，造成任意文件读取，不知道为啥失败了，环境：php5.3.29，**magic_quotes_gpc**=OFF

- 如果php版本小于5.2.8，linux 需要文件名长于 4096，windows 需要长于 256，超过部分会被丢弃从而实现文件包含绕过后缀.php限制





## 查看配置文件

通过install目录下的InstallLock.txt是否存在来判断是否安装，如果可以删除该文件，就可以配合入口文件的文件包含漏洞来重新安装网站。

![image-20230525135351792](../../../images/image-20230525135351792-1686141975685.png)

数据库编码为`utf-8`，不存在宽字节注入

![image-20230525135521039](../../../images/image-20230525135521039-1686141975685.png)





## 查看身份校验文件

![image-20230525140710113](../../../images/image-20230525140710113-1686141975685.png)

垂直越权，只要在cookie中加入user值

![image-20230525140957487](../../../images/image-20230525140957487-1686141975685.png)





## 前台

### `about.php`

![image-20230525141923350](../../../images/image-20230525141923350-1686141975685.png)

使用`addslashes`来过滤，逃逸不了引号，所以不存在sql注入



### `concat.php`

sql都用了`addslashes`来过滤

但是存在xss

![image-20230525142829890](../../../images/image-20230525142829890-1686141975685.png)



![image-20230525142841766](../../../images/image-20230525142841766-1686141975685.png)



![image-20230525143133231](../../../images/image-20230525143133231-1686141975685.png)



![image-20230525143813133](../../../images/image-20230525143813133-1686141975685.png)

```php
cookie:name="><script>alert(/xss/)</script>
```

![image-20230525144008677](../../../images/image-20230525144008677-1686141975686.png)







### `content.php`

存在数字型终于可以注入了，配合`mysql_error`进行报错注入

![image-20230525144109692](../../../images/image-20230525144109692-1686141975686.png)

![image-20230525144251129](../../../images/image-20230525144251129-1686141975686.png)





### `submit.php`

![image-20230525145929543](../../../images/image-20230525145929543-1686141975686.png)

![image-20230525151458927](../../../images/image-20230525151458927-1686141975686.png)

因为验证码正确后也不刷新，所以可以使用同一个验证码进行注入

![image-20230525151429571](../../../images/image-20230525151429571-1686141975686.png)







存储xss

过滤

![image-20230525145800880](../../../images/image-20230525145800880-1686141975686.png)

但是还是可以在name处保存

输出点

![image-20230525145901519](../../../images/image-20230525145901519-1686141975686.png)





## 后台

### 后台登录

![image-20230525153016604](../../../images/image-20230525153016604-1686141975686.png)

没有过滤，可以使用联合查询构造临时用户数据登录,或者报错注入拿到密码后解密登录

判断字段数

![image-20230525153002742](../../../images/image-20230525153002742-1686141975686.png)

确定用户名，密码字段位置

![image-20230525154135880](../../../images/image-20230525154135880-1686141975687.png)

联合查询登陆

```php
user=-1'+union+select+1,2,'admin','c4ca4238a0b923820dcc509a6f75849b',5,6,7,8#&password=1&login=yes
```

![image-20230525154322362](../../../images/image-20230525154322362-1686141975687.png)





### 文件上传处

`up.class.php`

白名单，文件大小检测，文件时间戳重命名,图片裁剪生成缩略图

没啥用，需要配合其他漏洞





# DedeCMS

进行功能点审计

## 全局总览

```php
入口文件主要有三个
前台
后台：dede/
会员：member/
```

全局函数
在项目根目录的index文件包含的/include/common.inc.php中
过滤的：

检查和注册外部提交的变量，`CheckRequest`

![image-20230526102737909](../../../images/image-20230526102737909-1686141975687.png)

文件上传的

![image-20230526102806414](../../../images/image-20230526102806414-1686141975687.png)

分页参数的

![image-20230526102852980](../../../images/image-20230526102852980-1686141975687.png)

函数集

![image-20230526102920759](../../../images/image-20230526102920759-1686141975687.png)



`include/filter.inc.php`

过滤不文明内容的😅，不是为了防御漏洞的



## 文件上传功能点

### ` /dede/archives_do.php`

![image-20230525201745708](../../../images/image-20230525201745708-1686141975687.png)



![image-20230525200159854](../../../images/image-20230525200159854-1686141975687.png)

实际上是调用了`upload.helper.php`中的`AdminUpload`方法

![image-20230525201545164](../../../images/image-20230525201545164-1686141975687.png)

![image-20230525201700590](../../../images/image-20230525201700590-1686141975687.png)

可以看到只对文件的MIME类型进行了校验,没有对文件后缀进行校验

再看一下全局防护`uploadsafe.inc.php`

![image-20230525201856657](../../../images/image-20230525201856657-1686141975688.png)

![image-20230525201949967](../../../images/image-20230525201949967-1686141975688.png)

虽然有黑名单，但是黑名单对管理员用户无效

![image-20230525202104709](../../../images/image-20230525202104709-1686141975688.png)

最后用`getimagesize`进行校验，可以添加文件头绕过。

综上所属，是管理员身份，只需要修改MIME和文件头即可绕过过滤。

![image-20230525191610086](../../../images/image-20230525191610086-1686141975688.png)



![image-20230525191553945](../../../images/image-20230525191553945-1686141975688.png)



### `/dede/media_add.php`

![image-20230525205532459](../../../images/image-20230525205532459-1686141975688.png)

跟上一个差不多，只校验MIME，不校验后缀，只是多了一步加水印的，所以传一个php后缀的图片马

![image-20230525205858924](../../../images/image-20230525205858924-1686141975688.png)





### `/dede/file_manage_control.php`

![image-20230525204329712](../../../images/image-20230525204329712-1686141975689.png)



![image-20230525210840452](../../../images/image-20230525210840452-1686141975688.png)

![image-20230525211553932](../../../images/image-20230525211553932-1686141975689.png)

没有对内容进行过滤，这里可以写🐎



## URL重定向

`plus/download.php`

![image-20230526100708534](../../../images/image-20230526100708534-1686141975688.png)

![image-20230526100724469](../../../images/image-20230526100724469-1686141975689.png)

![image-20230526101319599](../../../images/image-20230526101319599-1686141975690.png)

`$linkinfo`压根不存在，没有过滤，对url先进行base64加密，再进行url加密即可

![image-20230526101138188](../../../images/image-20230526101138188-1686141975689.png)





## 会员任意密码修改

先看一下会员中心的逻辑

`member/index.php`

校验用户是否登录，登录则进入会员个人中心

`member/index_do.php`

会员操作与`$fmdo`参数匹配

重置密码

```php
index_do.php?fmdo=user&dopost=xxx
```

重置密码逻辑在`resetpassword.php`中,大致逻辑如下

```php
if($dopost == "")
{
  include(dirname(__FILE__)."/templets/resetpassword.htm");
}
elseif($dopost == "getpwd")
{
  	// 找回密码第一步
}
else if($dopost == "safequestion")
{
  	// 密码问题判断
}
else if($dopost == "getpasswd")
{
  	// 找回密码第二步
}
```

重点关注密码问题判断

![image-20230526123219865](../../../images/image-20230526123219865-1686141975689.png)

使用了弱比较，如果用户没有设置密码问题，默认值如下



![image-20230526123041472](../../../images/image-20230526123041472-1686141975689.png)

所以可以构造`safequestion=0.0&safeanswer=`来进入该if语句

跟进 sn，最终是重定向到修改密码页面

![image-20230526123510601](../../../images/image-20230526123510601-1686141975689.png)



## 任意用户登录

看一下怎么处理的

`member/config.php`

![image-20230526130641724](../../../images/image-20230526130641724-1686141975689.png)

跟进

![image-20230526130848007](../../../images/image-20230526130848007-1686141975689.png)

![image-20230526132740089](../../../images/image-20230526132740089-1686141975689.png)

对得到的`M_ID`做了`intval`处理，最后根据`M_ID`的值从数据库中取出对应用户

跟进`getnum`

![image-20230526132102568](../../../images/image-20230526132102568-1686141975689.png)

跟进`getcookie`

![image-20230526131125230](../../../images/image-20230526131125230-1686141975689.png)

两个IF，第一个if判断Cookie中的DedeUserID和DedeUserID__ckMD5,如果都存在进入第二个if，校验cookie有效性，可以看到是通过加盐再加密的方式，所以按理说是无法伪造cookie的。

但是可以从`/member/index.php`可以来获取这个加密后的值

搜索`getcookie`得到

![image-20230526131657609](../../../images/image-20230526131657609-1686141975689.png)

跟踪`$last_vid`

![image-20230526132259147](../../../images/image-20230526132259147-1686141975690.png)

如果`$last_vid`为空，那么将`$uid`赋值给`$last_vid`，并作为参数传递给`PutCookie`函数

跟进`PutCookie`函数

![image-20230526132158862](../../../images/image-20230526132158862-1686141975690.png)

可以看到跟登录的`getcookie`加密方式一样。

总结一下：

登录

```php
用户的身份由DedeUserID的值来决定

校验：DedeUserID__ckMd5的值和md5($cfg_cookie_encode.DedeUserID)
```

访问`member/index.php?uid=xx`

```php
last_vid=uid

last_vid__ckMd5=md5($cfg_cookie_encode.uid)
```

所以让

```
DedeUserID=last_vid=uid
DedeUserID__ckMd5=last_vid__ckMd5
```

就可以绕过getcookie的校验，但是还有一个问题是`uid`指的是用户的`userid`，`DedeUserID`指的是用户的`mid`，所以要让`uid`为`mid`才可以伪造身份

![image-20230526134001725](../../../images/image-20230526134001725-1686141975691.png)

还记得上面的处理吗？

![image-20230526134147020](../../../images/image-20230526134147020-1686141975690.png)

所以只需要构造一个含有数字1的字符串就可以伪造admin的身份。

注册一个用户名为1admin的用户

![image-20230526125711799](../../../images/image-20230526125711799-1686141975690.png)

```
DedeUserID=last_vid=uid
DedeUserID__ckMd5=last_vid__ckMd5
```

![image-20230526125702071](../../../images/image-20230526125702071-1686141975690.png)





# ThinkPHP

## 安装

```powershell
composer create-project --prefer-dist topthink/think=5.0.10 tp5.0.10
```

将 [composer](https://so.csdn.net/so/search?q=composer&spm=1001.2101.3001.7020).json 文件的 require 字段设置成如下

```php
"require": {
    "php": ">=5.4.0",
    "topthink/framework": "5.0.10"
}
```

更新

```powershell
composer update
```

将其放置到网站目录下，访问http://127.0.0.1/tp5.0.10/public/

![image-20230529155309821](../../../images/image-20230529155309821-1686141975690.png)



## 概览

推荐阅读https://www.cnblogs.com/yokan/p/16102644.html

### 路由和参数传递

```php
http://servername/index.php/模块/控制器/操作/[参数名/参数值...] # pathinfo模式

http://servername/index.php?s=/index/Index/index    # 兼容方式
```

```php
?name=213
/name/123
```

如控制器Index：`application/index/controller/Index.php`

![image-20230531232416306](../../../images/image-20230531232416306.png)

```php
http://127.0.0.1/tp5.0.10/public/index.php/index/Index/hello?name=world
http://127.0.0.1/tp5.0.10/public/index.php/index/Index/hello/name/world

http://127.0.0.1/tp5.0.10/public/index.php?s=/index/Index/hello/name/world
http://127.0.0.1/tp5.0.10/public/index.php?s=/index/Index/hello&name=world
```





## Request类任意调用__construct方法导致的rce

### 概述

Request核心类**$method** 来自可控的 **$_POST** 数组，而且在获取之后没有进行任何检查，直接把它作为 **Request** 类的方法进行调用，同时，该方法传入的参数是可控数据 **$_POST** 。导致可以随意调用 **Request** 类的部分方法

> **过程：**
>
> 让method等于 `__construct`魔术方法，然后里面的 `foreach`函数造成变量覆盖。然后通过**Request** 类中的 `param`方法最终又调用了`filterValue`方法，而该方法中就存在可利用的 **call_user_func** 函数，从而执行任意命令

> **Request** 类中的 `param、route、get、post、put、delete、patch、request、session、server、env、cookie、input` 方法均调用了 **filterValue** 方法，而该方法中就存在可利用的 **call_user_func** 函数

以5.0.10版本为例

payload

```http
http://127.0.0.1/tp5.0.10/public/index.php

# post
_method=__construct&filter=system&cmd=whoami
```

![image-20230531225820851](../../../images/image-20230531225820851.png)



### 调试

第一个断点下在`thinkphp/libary/think/App.php`文件，调用`routeCheck`进行调度解析这里

![image-20230531221156861](../../../images/image-20230531221156861.png)

跟进`routeCheck`

![image-20230531221218359](../../../images/image-20230531221218359.png)

跟进`check`

![image-20230531221233596](../../../images/image-20230531221233596.png)

跟进`method`

![image-20230531221259178](../../../images/image-20230531221259178.png)

`var_method`在`application/config.php`中

![image-20230531222320838](../../../images/image-20230531222320838.png)

所以这里可以控制`method`变量，从而任意调用Request类的方法。

如果调用`__construct`方法

![image-20230531221707200](../../../images/image-20230531221707200.png)

存在一个`foreach`循环，如果传入的`options`数组的键名为该类的属性，就用键值覆盖该属性的值。

![image-20230531222852250](../../../images/image-20230531222852250.png)

继续往下

在`App::run()`方法里面，如果我们开启了debug模式，则会调用`Request::param()`方法：

![image-20230531223011915](../../../images/image-20230531223011915.png)

就算没有开启debug模式，下面的exec方法也会调用

![image-20230531230604547](../../../images/image-20230531230604547.png)

![image-20230531220640414](../../../images/image-20230531220640414.png)

跟进`Request::param()`方法

将获取到的post参数用`array_merge`与get方式的参数进行合并

![image-20230531225110758](../../../images/image-20230531225110758.png)

最后将其传入`input`中

![image-20230531224212939](../../../images/image-20230531224212939.png)

跟进`Request::input()`,`array_walk_recursive`  对数组中的每个成员递归地应用用户函数

![image-20230531224435122](../../../images/image-20230531224435122.png)

然后`filterValue`方法中，调用了`call_user_func`造成任意命令执行

![image-20230531223158698](../../../images/image-20230531223158698.png)

### 关键点总结

From：七月火师傅的一张流程图

![img](../../../images/1964477-20220405155310200-1040131440.png)

### 其他版本

流程大同小异

payload总结：https://y4er.com/posts/thinkphp5-rce/#thinkphp5-method%E4%BB%BB%E6%84%8F%E8%B0%83%E7%94%A8%E6%96%B9%E6%B3%95%E5%AF%BC%E8%87%B4rce





