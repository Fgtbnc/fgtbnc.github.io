---
title: Vulnhub--blue系列
date: 2023/6/1 19:48:25
categories:
- 渗透测试
tags:
- 靶场
---



### 1

<img src="../../../images/image-20230305123445338.png" alt="image-20230305123445338" style="zoom:80%;" />

![image-20230305130206056](../../../images/image-20230305130206056-1686283466992.png)

![image-20230305130221501](../../../images/image-20230305130221501-1686283466992.png)

secret.zip没有破解出来。

dirseach目录扫描结果

![image-20230305130651336](../../../images/image-20230305130651336-1686283466992.png)

访问robots.txt中的网址

![image-20230305130727097](../../../images/image-20230305130727097-1686283466992.png)

歌词连在一起MD5为ssh密码，没成功。

另一个网站为/etc/dripispowerful.html，猜测有文件包含，根据上面的目录扫描只有一个index.php，参数尝试压缩包里的focus  on "drip"。

尝试访问index.php?drip=/etc/dripispowerful.html，获得密码和两个用户名

![image-20230305131636950](../../../images/image-20230305131636950-1686283466992.png)

ssh登录

![image-20230305132010055](../../../images/image-20230305132010055-1686283466992.png)

提权（不会，看wp的）

![image-20230305133355153](../../../images/image-20230305133355153-1686283466992.png)

polkit提权https://github.com/Almorabea/Polkit-exploit

![image-20230305133346301](../../../images/image-20230305133346301-1686283466992.png)

![image-20230305133331628](../../../images/image-20230305133331628-1686283466992.png)





### 2

- ftp匿名登录
- wordpress用户爆破，后台getshell
- ssh密钥登录
- nmap提权

![image-20230304120228644](../../../images/image-20230304120228644-1686283466993.png)

![image-20230304120308755](../../../images/image-20230304120308755-1686283466993.png)

ftp存在匿名登录，经过验证发现ftp上的secret.jpg无隐写。

![image-20230304120324693](../../../images/image-20230304120324693-1686283466993.png)



![image-20230304120358152](../../../images/image-20230304120358152-1686283466993.png)



wpscan用户枚举

![image-20230304121310981](../../../images/image-20230304121310981-1686283466993.png)

进行密码爆破

![image-20230304122015951](../../../images/image-20230304122015951-1686283466993.png)



访问后台时跳转，添加host记录

![image-20230304121229221](../../../images/image-20230304121229221-1686283466993.png)

进入后台

在主题的404文件中加入一句话木马，访问该404文件，传入反弹shell

![image-20230304125406319](../../../images/image-20230304125406319-1686283466993.png)

![image-20230304125413838](../../../images/image-20230304125413838-1686283466993.png)

![image-20230304125353482](../../../images/image-20230304125353482-1686283466993.png)

进入家目录看看

![image-20230304125605846](../../../images/image-20230304125605846-1686283466993.png)

无权限，尝试提权失败

但是还存在ssh密钥，我们可以将ssh私钥下载到本地，通过ssh登录到freddie用户

使用python在.ssh目录下开一个简易服务器供本地下载文件。

![image-20230304131800088](../../../images/image-20230304131800088-1686283466993.png)

本地下载下来后，需要将id_rsa改为只有400权限才可以使用

![image-20230304132002887](../../../images/image-20230304132002887-1686283466993.png)

![image-20230304132104344](../../../images/image-20230304132104344-1686283466993.png)



![image-20230304132257002](../../../images/image-20230304132257002-1686283466993.png)

![image-20230304132307798](../../../images/image-20230304132307798-1686283466993.png)

![image-20230304132242555](../../../images/image-20230304132242555-1686283466993.png)

读取/root/root.txt获得第二个flag·

![image-20230304132345766](../../../images/image-20230304132345766-1686283466993.png)

### 3

- ssh日志包含getshell
- ssh密钥登录
- 命令劫持提权

![image-20230304141715650](../../../images/image-20230304141715650-1686283466994.png)

![image-20230304141957453](../../../images/image-20230304141957453-1686283466994.png)

几个网站都是ABAB。。

wp-admin目录只有一个readme

![image-20230304141834709](../../../images/image-20230304141834709-1686283466994.png)

恶作剧😡

回头看namp的默认脚本的扫描，可以看到robots.txt中禁止了一个/eventadmins，访问

![image-20230304142524510](../../../images/image-20230304142524510-1686283466994.png)

再访问提示ctrl+a发现

![image-20230304142655946](../../../images/image-20230304142655946-1686283466994.png)

![image-20230304142745118](../../../images/image-20230304142745118-1686283466994.png)

再访问

![image-20230304142923226](../../../images/image-20230304142923226-1686283466994.png)

一开始想的是用九头蛇进行爆破，但是用户只知道root。

后面发现不行，想到日志的用途，尝试进行ssh日志包含getshell。

因为日志中会记录ssh连接的用户名

![image-20230304165054436](../../../images/image-20230304165054436-1686283466994.png)

所以

![image-20230304165131376](../../../images/image-20230304165131376-1686283466994.png)

反弹shell

![image-20230304165152817](../../../images/image-20230304165152817-1686283466994.png)

![image-20230304165204015](../../../images/image-20230304165204015-1686283466994.png)

这里和上一个靶机一样，存在另一个用户robertj，并且www-data无法提权

按照上一个的思路查看.ssh目录，发现为空

![image-20230304165345630](../../../images/image-20230304165345630-1686283466994.png)

选择直接生成ssh密钥

<img src="../../../typora img/image-20230304175028256.png" alt="image-20230304175028256" style="zoom:80%;" />

查看ssh配置文件：只允许密钥验证方式连接，并且用来验证的公钥为authorized_keys

![image-20230304171108734](../../../images/image-20230304171108734-1686283466994.png)

所以需要将公钥id_rsa.pub改为authorized_keys

然后用python在靶机上开一个http服务器，kali使用wget命令下载私钥，并修改私钥权限，最后ssh登录

![image-20230304175249187](../../../images/image-20230304175249187-1686283466994.png)

提权

![image-20230304174502247](../../../images/image-20230304174502247-1686283466994.png)

一开始想用strings查看的，但是没有安装，那就直接用一下看看

![image-20230304174629368](../../../images/image-20230304174629368-1686283466994.png)

可以知道这个命令实质上是执行了

```shell
ip address
cat /etc/hosts
uname -a
```

那么我们选一个进行构造即可

```shell
echo "/bin/bash" > uname
chmod 777 uname 
export PATH=/home/robertj:$PATH
```

然后执行getinfo即可

![image-20230304174854741](../../../images/image-20230304174854741-1686283466995.png)

### 4

- misc获得人员信息
- ftp爆破
- ftp+sync服务，导致可以通过ftp服务器操作网站服务器目录
- ssh密钥登录
- 命令劫持提权

![image-20230304191909931](../../../images/image-20230304191909931-1686283466995.png)

网站注释解码，访问另一个网站，brainfuck解码，二维码扫描最后得到如下

![image-20230304204625662](../../../images/image-20230304204625662-1686283466995.png)

收集到了网站技术人员的名字，可以尝试爆破ftp和ssh服务。

```
luther
gary
hubert
clark
```

ssh无果

ftp

```
login: luther   password: mypics
```

![image-20230304210341290](../../../images/image-20230304210341290-1686283466995.png)

![image-20230304211248271](../../../images/image-20230304211248271-1686283466995.png)

发现了一个目录，目录名为hubert是收集到的一个用户，并且uid和gid为1001，证明其是第一个用户，所以该目录为hubert用户的家目录。

另一个文件使用get命令下载到本地后

![image-20230304212449213](../../../images/image-20230304212449213-1686283466995.png)

可以发现是一个同步完成的日志。

> 所以根据这两个信息，我们可以知道服务器会同步这个ftp上的文件，就相当于我们可以操作hubert用户的家目录。

我们使用put命令上传自己的公钥

![image-20230304212132365](../../../images/image-20230304212132365-1686283466995.png)

需要注意公钥名

![image-20230304212209119](../../../images/image-20230304212209119-1686283466995.png)

然后ssh连接

![image-20230304212235168](../../../images/image-20230304212235168-1686283466995.png)

看下家目录有啥

![image-20230304212842826](../../../images/image-20230304212842826-1686283466995.png)

读取这个root权限的py文件，我们可以知道这个网站被黑客frica攻击了，并且留下了后门，所以这也是为什么前面叫人来修复网站的原因🤣

这个黑客还告诉我们他留下了松散的权限😍

![image-20230304212825018](../../../images/image-20230304212825018-1686283466995.png)

让我康康

![image-20230304213424178](../../../images/image-20230304213424178-1686283466995.png)

这次有strings命令，查看getinfo

![image-20230304213618067](../../../images/image-20230304213618067-1686283466995.png)

跟上一个靶机一样

![image-20230304213721571](../../../images/image-20230304213721571-1686283466995.png)

### 5

![image-20230305160546454](../../../images/image-20230305160546454-1686283466995.png)

![image-20230305163954224](../../../images/image-20230305163954224-1686283466995.png)

wpscan用户枚举

![image-20230305160724611](../../../images/image-20230305160724611-1686283466995.png)

```
abuzerkomurcu
collins
gill
collins
satanic
```

密码字典生成

![image-20230305162409527](../../../images/image-20230305162409527-1686283466995.png)

> -m 6 是因为wordpress密码最少6个字符

爆破密码

![image-20230305162556601](../../../images/image-20230305162556601-1686283466995.png)

```
gill / interchangeable
```

普通用户，后台只找到一张可以图片DB

![image-20230305164352862](../../../images/image-20230305164352862-1686283466996.png)

分析这张图片

![image-20230305164731337](../../../images/image-20230305164731337-1686283466996.png)

![image-20230305164827742](../../../images/image-20230305164827742-1686283466996.png)

得到ssh账号

```
gill 59583hello
```

登陆后发现

![image-20230305165100247](../../../images/image-20230305165100247-1686283466996.png)

```
lost+found目录无权限

家目录下有一个kEYFILE
```

通过搜索得到

![image-20230305165731867](../../../images/image-20230305165731867-1686283466996.png)

![image-20230305171048832](../../../images/image-20230305171048832-1686283466996.png)

可以使用john进行破解，机子GPU太垃圾了，直接从wp拿到密码

获得密码后，可以用KeePass软件https://sourceforge.net/projects/keepass/打开，也可以用在线网站

https://app.keeweb.info/打开。

打开后发现6个空白的key

![image-20230305172848464](../../../images/image-20230305172848464-1686283466996.png)

不知道拿来干嘛的。。

继续信息收集，发现根目录下有一个keyfolder，可能就是要让上面的key放在这个keyfolder

而且使用pyps64监控进程发现有一个定时任务key.sh，每一分钟执行一次。

![image-20230305172941445](../../../images/image-20230305172941445-1686283466996.png)

经测试当只有一个文件fracturedocean时

![image-20230305174849955](../../../images/image-20230305174849955-1686283466996.png)

![image-20230305175028644](../../../images/image-20230305175028644-1686283466996.png)

定时脚本内容

![image-20230305175728426](../../../images/image-20230305175728426-1686283466996.png)



### 6

![image-20230307141431844](../../../images/image-20230307141431844-1686283466996.png)

只开放了80端口

![image-20230307141917191](../../../images/image-20230307141917191-1686283466996.png)

提示我们目录扫描要添加zip类型，说明网站目录应该存在zip文件

目录扫描

dirserach

![image-20230307142511029](../../../images/image-20230307142511029-1686283466996.png)

gobuster

![image-20230307142447064](../../../images/image-20230307142447064-1686283466996.png)

访问/spammer下载得到spammer.zip

使用john爆破

![image-20230307142628410](../../../images/image-20230307142628410-1686283466997.png)

解压得到

```
mayer:lionheart
```

访问网站

![image-20230307141958670](../../../images/image-20230307141958670-1686283466997.png)

尝试登录解压得到的用户，成功登录

![image-20230307142917723](../../../images/image-20230307142917723-1686283466997.png)

逛了一圈发现，页面编辑，邮箱泄露，插件（无法加载作罢）。

- 网站配置

  ```
  Textpattern version: 4.8.3 (596bca03a4b32004412499363cecec62)
  Last update: 2020-09-13 19:56:06
  Site URL: 192.168.2.35/textpattern
  Admin URL: 192.168.2.35/textpattern/textpattern
  Document root: /var/www
  $path_to_site: /var/www/textpattern
  Textpattern path: /var/www/textpattern/textpattern
  Article URL pattern: messy
  Production status: testing
  Temporary directory path: /tmp
  PHP version: 5.5.38-1~dotdeb+7.1
  GD Graphics Library: Unavailable
  Server timezone: UTC
  Server local time: 2023-03-07 14:40:29
  Daylight Saving Time enabled?: 0
  Automatically adjust Daylight Saving Time setting?: 1
  Time zone (GMT offset in seconds): Asia/Baghdad (10800)
  MySQL: 5.5.47-0+deb7u1 ((Debian)) 
  Database server time: 2023-03-07 08:40:29
  Database server time offset: 0 s
  Database server timezone: SYSTEM
  Database session timezone: SYSTEM
  Locale: C
  Site / Admin language: en / en
  Web server: Apache/2.2.22 (Debian)
  Apache version: Apache/2.2.22 (Debian)
  PHP server API: apache2handler
  RFC 2616 headers: 
  Server OS: Linux 3.2.0-4-amd64
  Admin-side theme: hive 4.8.3
  
  Pre-flight check: 
  ------------------------
  
  New Textpattern version 4.8.8 available for download. Help
  
  DNS lookup failed: 192.168.2.35 Help
  
  /var/www/textpattern/textpattern/setup/ still exists. Help
  
  Site URL preference might be incorrect: 192.168.174.159/textpattern Help
  
  Image directory is not writable: /var/www/textpattern/images
  Theme directory is not writable: /var/www/textpattern/themes
  Plugin directory is not writable: /var/www/textpattern/textpattern/plugins Help
  ------------------------
  
  .htaccess file contents: 
  ------------------------
  # BEGIN Textpattern
  
  #DirectoryIndex index.php index.html
  
  <IfModule mod_rewrite.c>
      RewriteEngine On
  
      # Enable the `FollowSymLinks` option below if it isn't already.
      #Options +FollowSymlinks
  
      #RewriteBase /relative/web/path/
  
      RewriteCond %{REQUEST_FILENAME} -f [OR]
      RewriteCond %{REQUEST_FILENAME} -d
      RewriteRule ^(.+) - [PT,L]
  
      RewriteCond %{REQUEST_URI} !=/favicon.ico
      RewriteRule ^(.*) index.php
  
      RewriteCond %{HTTP:Authorization}  !^$
      RewriteRule .* - [E=REMOTE_USER:%{HTTP:Authorization}]
  </IfModule>
  
  <IfModule mod_mime.c>
      AddType image/svg+xml  svg svgz
      AddEncoding gzip       svgz
  </IfModule>
  
  # For additional Apache-compatible web server configuration settings to enhance
  # site performance and security, we recommend:
  # https://github.com/h5bp/server-configs-apache/blob/master/dist/.htaccess
  
  # END Textpattern
  
  ------------------------
  ```

- 文件上传

  可以直接上传🐎，无任何防护

  漏洞分析：https://blog.csdn.net/yun2diao/article/details/92765372

  反弹shell

  <img src="../../../typora img/image-20230307151417930.png" alt="image-20230307151417930" style="zoom:80%;" />



提权

上传linepeas.sh查找可提权项

这里查找CVE

![image-20230307153044644](../../../images/image-20230307153044644-1686283466997.png)

选择一个cve进行提权，我这里选择的是脏牛提权https://github.com/firefart/dirtycow

![image-20230307154757476](../../../images/image-20230307154757476-1686283466997.png)

![image-20230307154831929](../../../images/image-20230307154831929-1686283466997.png)

### 7

![image-20230307182708576](../../../images/image-20230307182708576-1686283466997.png)

![image-20230307182726635](../../../images/image-20230307182726635-1686283466997.png)

开放了很多端口和服务，不同端口上都有网站，所以目录扫描时需要注意扫哪个端口

```
66
80
8086
```

- 80端口

  ![image-20230307224547682](../../../images/image-20230307224547682-1686283466997.png)

  msf一条龙

![image-20230307224427084](../../../images/image-20230307224427084-1686283466997.png)

![image-20230307224412649](../../../images/image-20230307224412649-1686283466997.png)



- 66端口

  dirsearch扫描结果

  ![image-20230307224912102](../../../images/image-20230307224912102-1686283466997.png)

  ![image-20230307224854194](../../../images/image-20230307224854194-1686283466998.png)

  ​	历史命令操作.bash_history重要部分

  ```
  wget 192.168.2.43:81/root.txt
  mv root.txt flag.txt
  nano flag.txt
  ```

  可以发现从别的主机上复制了flag，并重名为flag.txt

- 8086

  ![image-20230307225845372](../../../images/image-20230307225845372-1686283466998.png)

  都是静态页面



这里看了wp正常是从66端口目录扫描出/eno目录，下载并破解zip压缩包，获得80端口的用户凭证的。。





### 9

不知道为什么没有8

![image-20230307232027664](../../../images/image-20230307232027664-1686283466998.png)

网站

只有一个登录框

sql注入失败，弱口令爆破失败



目录扫描

![image-20230307234601067](../../../images/image-20230307234601067-1686283466998.png)

看到backup还以为有备份文件可以进行代码审计，但是没有；

访问/admin/home.php尝试是否存在未授权，发现没有。



寻找网站框架漏洞 

![image-20230307234835760](../../../images/image-20230307234835760-1686283466999.png)

尝试了之后发现只存在rce漏洞

![image-20230307233410737](../../../images/image-20230307233410737-1686283466999.png)

![image-20230307233603248](../../../images/image-20230307233603248-1686283466999.png)

![image-20230307233643324](../../../images/image-20230307233643324-1686283466999.png)

使用第二个py脚本

![image-20230308001806203](../../../images/image-20230308001806203-1686283466999.png)

![image-20230308001820794](../../../images/image-20230308001820794-1686283466999.png)

这个脚本很贴心地给出了服务器信息，并读取了配置文件，获得了clapton用户的凭证。

![image-20230307234930848](../../../images/image-20230307234930848-1686283467000.png)

发现大部分命令明明可以使用的却都执行不了，所以需要反弹shell到kali，经测试可以使用nc进行反弹shell

![image-20230308130950848](../../../images/image-20230308130950848-1686283467000.png)

使用python切换为交互式shell，切换到clapton用户

![image-20230308131017546](../../../images/image-20230308131017546-1686283467000.png)

发现clapton家目录下有note.txt

![image-20230308131125354](../../../images/image-20230308131125354-1686283467000.png)

读取

![image-20230308131243068](../../../images/image-20230308131243068-1686283467000.png)

提示我们用缓冲区溢出漏洞提权，这部分pwn不会

按照下面文章中的方法进行复现

https://zhuanlan.zhihu.com/p/570218595

最后贴上复现成功的截图

![image-20230308130938122](../../../images/image-20230308130938122-1686283467000.png)