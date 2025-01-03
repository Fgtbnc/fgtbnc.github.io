---
title: bugku 应急响应
date: 2023/5/31 19:52:40
categories:
- CTF
tags:
- 靶场
---



### js劫持

![image-20230402174621652](../images/image-20230402174621652-1686151030355.png)

```
cat `find /var/www/html -name "6127418cad73c.php"`
```

![image-20230402174633296](../images/image-20230402174633296-1686151030358.png)

```shell
find . | xargs grep -ri '<script type="text/javascript">' -l | sort | uniq -c

```

通过xss的特征内容来定位可疑文件



### 反弹shell后门

![image-20230402180138838](../images/image-20230402180138838-1686151030359.png)

![image-20230402180201438](../images/image-20230402180201438-1686151030359.png)

处理：

删除文件

杀掉进程



### 命令劫持

![image-20230402180338299](../images/image-20230402180338299-1686151030359.png)



### 数据库修复

[MySQL之权限管理 - I’m Me! - 博客园](https://www.cnblogs.com/richardzhu/p/3318595.html)

![image-20230402181706798](../images/image-20230402181706798-1686151030359.png)

```mysql
revoke file on *.* from 'root'@'localhost'; # 收回文件权限
set global general_log = off; # 不启用日志
flush privileges; # 刷新权限
```



### 用户删除

![image-20230402183352852](../images/image-20230402183352852-1686151030359.png)

直接删就完事了



### 首次攻击

筛选了ip和状态码，然后找的。

![image-20230402190333789](../images/image-20230402190333789-1686151030360.png)





![image-20230403114840046](../images/image-20230403114840046-1686151030360.png)

![image-20230403114851126](../images/image-20230403114851126-1686151030360.png)

发现安装了phpmyadmin，那么很大概率是通过数据库攻入的。

![image-20230403115435870](../images/image-20230403115435870-1686151030360.png)

登录phpmyadmin后台查看二进制日志，很明显看出来这个是使用udf创建恶意函数sys_eval提权

```sql
# 查看这些目录是否有文件，很可能有提权
mysql\lib\plugin

c:/windows/system32/wbem/mof/
```

![image-20230403115606245](../images/image-20230403115606245-1686151030360.png)

修复

![image-20230403123447079](../images/image-20230403123447079-1686151030360.png)

在my.ini中添加如上设置，不允许导入和导出。





进行mysql日志分析，通过定位sys_eval来查找攻击者进行了哪些操作

![image-20230403120034903](../images/image-20230403120034903-1686151030360.png)

只进行了添加用户的操作

![image-20230403120253606](../images/image-20230403120253606-1686151030360.png)

直接删除该用户

![image-20230403120327311](../images/image-20230403120327311-1686151030360.png)

查杀websehll

使用日志分析工具

（跟踪ip的访问）

![image-20230403122636304](../images/image-20230403122636304-1686151030360.png)

![image-20230403122604222](../images/image-20230403122604222-1686151030360.png)



后门查杀

![image-20230403124818101](../images/image-20230403124818101-1686151030361.png)

![image-20230403124754804](../images/image-20230403124754804-1686151030361.png)

木马位置在启动项

![image-20230403124903028](../images/image-20230403124903028-1686151030361.png)

![image-20230403124408821](../images/image-20230403124408821-1686151030361.png)

通过资源监视器得到该木马外联地址