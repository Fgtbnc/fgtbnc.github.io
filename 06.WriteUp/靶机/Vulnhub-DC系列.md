---
title: Vulnhub--DC系列
date: 2023/6/1 19:48:25
categories:
- 渗透测试
tags:
- 靶场
---



## DC1

#### 主机发现

```shell
sudo netdiscover 192.168.174.0/24
```

![image-20230223133100570](../../../images/image-20230223133100570-1686283446321.png)

![image-20230223133203500](../../../images/image-20230223133203500-1686283446321.png)

#### 端口扫描

![image-20230223131655833](../../../images/image-20230223131655833-1686283446321.png)

#### 查看网站

![image-20230223133357605](../../../images/image-20230223133357605-1686283446321.png)

只有注册，登录，找回密码功能（这里用sqlmap没有成功）

很容易知道其是Drupal系统。

#### 寻找漏洞

![image-20230223133730288](../../../images/image-20230223133730288-1686283446321.png)

这里优先使用日期较新，等级为优秀的exp（也可以通过收集到的系统版本寻找日期）

![image-20230223133945118](../../../images/image-20230223133945118-1686283446321.png)

发现成功了。

#### 获得shell后的信息收集

![image-20230223134100934](../../../images/image-20230223134100934-1686283446322.png)

网上查找可知Drupal的配置文件在` /sites/default/settings.php`,读取后得到flag2和数据库配置信息

![img](../../../images/1962254-20200713224317160-859515027-1686283446322.png)

连接数据库，需要先将shell切换为交互式

```shell
python -c 'import pty;pty.spawn("/bin/bash")'
```

![image-20230223134424820](../../../images/image-20230223134424820-1686283446322.png)

连接后，查询用户表收集管理员账户信息。

![image-20230223130117676](../../../images/image-20230223130117676-1686283446322.png)

可以看到这里admin的密码是加密了的。这里有三种思路

- 暴力破解

  > 使用john失败了。

  ![image-20230223135625792](../../../images/image-20230223135625792-1686283446323.png)

- 修改密码

  > 需要找到对应的加密脚本。

  ![image-20230223135430041](../../../images/image-20230223135430041-1686283446323.png)

  ![image-20230223130607081](../../../images/image-20230223130607081-1686283446323.png)

  

  ![image-20230223131033472](../../../images/image-20230223131033472-1686283446323.png)

- 增加一名用户（管理员权限）

  使用[SearchSploit](https://blog.csdn.net/whatday/article/details/102806149)查找是否有对应的脚本

  ![image-20230223142047059](../../../images/image-20230223142047059-1686283446323.png)

  ![image-20230223142453365](../../../images/image-20230223142453365-1686283446323.png)

  ```shell
  python2  /usr/share/exploitdb/exploits/php/webapps/34992.py -t http://192.168.174.142/ -u admin1 -p 12345
  ```

  ![image-20230223142640254](../../../images/image-20230223142640254-1686283446323.png)

  ![image-20230223142651974](../../../images/image-20230223142651974-1686283446323.png)

### 登录后

![image-20230223131339443](../../../images/image-20230223131339443-1686283446323.png)



![image-20230223131415615](../../../images/image-20230223131415615-1686283446323.png)



![image-20230223131557531](../../../images/image-20230223131557531-1686283446323.png)

![image-20230223131846299](../../../images/image-20230223131846299-1686283446324.png)

尝试提权

![image-20230223131936343](../../../images/image-20230223131936343-1686283446324.png)

![image-20230223131903800](../../../images/image-20230223131903800-1686283446324.png)



![image-20230223131922331](../../../images/image-20230223131922331-1686283446324.png)



![image-20230223132056437](../../../images/image-20230223132056437-1686283446324.png)

![image-20230223132116100](../../../images/image-20230223132116100-1686283446324.png)

```
$6$Nk47pS8q$vTXHYXBFqOoZERNGFThbnZfi5LN0ucGZe05VMtMuIFyqYzY/eVbPNMZ7lpfRVc0BYrQ0brAhJoEzoEWCKxVW80
```

![image-20230223132610089](../../../images/image-20230223132610089-1686283446324.png)

破解成功后登录flag4用户

![image-20230223132741851](../../../images/image-20230223132741851-1686283446324.png)

![image-20230223132857455](../../../images/image-20230223132857455-1686283446324.png)

仍然使用上面的find提权

![image-20230223132915683](../../../images/image-20230223132915683-1686283446324.png)



## DC2

https://cloud.tencent.com/developer/article/1801074

- ip重定向，需要添加host记录
- cewl密码字典生成，密码爆破
- 7744端口的ssh爆破
- rbash限制
- git提权



## DC3

https://blog.csdn.net/bwt_D/article/details/121291921

- 信息收集：得到网站CMS，使用对应版本漏洞exp爆库
- john密码哈希爆破
- 后台任意文件上传
- 内核提权



## DC4

https://blog.csdn.net/weixin_44288604/article/details/108018008

- 用户登录无防护爆破
- rce
- ssh爆破
- 信息收集：备份文件，邮件（泄露用户密码）
- teehee提权

![image-20230224161511140](../../../images/image-20230224161511140-1686283446324.png)



## DC5

https://www.freebuf.com/sectool/259277.html

- 日志包含getshell
- screen提权

端口扫描

<img src="../../../typora img/image-20230224202703382.png" alt="image-20230224202703382" style="zoom: 80%;" />

通过contact提交后，页面返回的页脚不同判断出后端有include函数包含了页脚文件。

![image-20230224204544733](../../../images/image-20230224204544733-1686283446324.png)

![image-20230224204559624](../../../images/image-20230224204559624-1686283446325.png)

使用burp爆破参数

![image-20230224204633133](../../../images/image-20230224204633133-1686283446325.png)

<img src="../../../typora img/image-20230224204657204.png" alt="image-20230224204657204" style="zoom:80%;" />

文件包含getshell

因为网站没有文件上传功能，所以可以考虑包含日志文件/session文件，或者与phpinfo界面连用。

这里选择包含日志文件



## DC6

https://blog.csdn.net/weixin_45996361/article/details/123431118

- 用户密码爆破（字典为DC2的）
- wordpress插件activity monitor提供了ping命令，对用户输入无限制导致rce
- 邮件密码泄露
- sudo提权
- nmap提权

![image-20230225172831079](../../../images/image-20230225172831079-1686283446325.png)

![image-20230225202530004](../../../images/image-20230225202530004-1686283446325.png)

![image-20230225202604851](../../../images/image-20230225202604851-1686283446325.png)

![image-20230225202617319](../../../images/image-20230225202617319-1686283446325.png)



![image-20230225202517080](../../../images/image-20230225202517080-1686283446325.png)

![image-20230225202502557](../../../images/image-20230225202502557-1686283446325.png)

## DC7

- 端口：22,80

- 信息收集：github源码泄露

  ![image-20230225211326780](../../../images/image-20230225211326780-1686283446325.png)

  <img src="../../../typora img/image-20230225211258444.png" alt="image-20230225211258444" style="zoom:80%;" />

  ![image-20230225211553018](../../../images/image-20230225211553018-1686283446325.png)

  这个账户测试后可以连接ssh

- ssh连接后信息收集

  有邮件

  ![image-20230225211951236](../../../images/image-20230225211951236-1686283446325.png)

  

  ![image-20230225211719078](../../../images/image-20230225211719078-1686283446326.png)

  发现都是以下信息，并且每隔一段时间都有`You have new mail`提示，可以猜测出这是一个定时脚本

  ![image-20230225211729539](../../../images/image-20230225211729539-1686283446326.png)

  ![image-20230225211814513](../../../images/image-20230225211814513-1686283446326.png)

  ​	

```shell
dc7user@dc-7:~$ cat /opt/scripts/backups.sh 
#!/bin/bash
rm /home/dc7user/backups/*
cd /var/www/html/
drush sql-dump --result-file=/home/dc7user/backups/website.sql
cd ..
tar -czf /home/dc7user/backups/website.tar.gz html/
gpg --pinentry-mode loopback --passphrase PickYourOwnPassword --symmetric /home/dc7user/backups/website.sql
gpg --pinentry-mode loopback --passphrase PickYourOwnPassword --symmetric /home/dc7user/backups/website.tar.gz
chown dc7user:dc7user /home/dc7user/backups/*
rm /home/dc7user/backups/website.sql
rm /home/dc7user/backups/website.tar.gz
```

drush是drupal的命令，可以修改admin密码 ，不知道为什么在网站目录下才可以

gpg是公钥加密算法，没找到公钥，所以无法解密，否则可以解密重新导入sql文件

`mysql -udc7user -pMdR3xOgB7#dW Staff < /home/dc7user/backups/website.sql;`

- 提权

  ​	通过修改admin密码登录后台，可以编辑博客或者页面。那么直接选择编辑页面写入一个木马。但是需要先安装PHP模块。

  ​	木马写入后，获得www-data用户权限

  ​	![image-20230225225209155](../../../images/image-20230225225209155-1686283446326.png)

  从定时脚本所属的用户和用户组入手，用户组为www-data权限为rwx：说明www

  -data用户可以对其进行读写执行，拥有者为root：说明其运行时的权限为root。所以我们向其写入反弹shell，等待其执行就可以获得root权限。

  




## DC8

https://blog.csdn.net/q90375412/article/details/127351747

- sql注入（sqlmap）
- john破解密码
- 后台模板getshell
- exim4提权





## DC9

https://blog.csdn.net/m0_65712192/article/details/129250059

![image-20230309113640488](../../../images/image-20230309113640488-1686283446326.png)

可以发现ssh服务是filtered的

![image-20230309115117489](../../../images/image-20230309115117489-1686283446326.png)



![image-20230309115144629](../../../images/image-20230309115144629-1686283446326.png)

![image-20230309121325288](../../../images/image-20230309121325288-1686283446326.png)





![image-20230309121309508](../../../images/image-20230309121309508-1686283446326.png)

![image-20230309121245679](../../../images/image-20230309121245679-1686283446327.png)