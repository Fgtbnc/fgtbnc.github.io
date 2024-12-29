---
title: Linux常用命令
date: 2023/5/31 19:48:25
categories:
- Linux
---



Linux一切皆文件

没有信息就是好消息，说明命令成功了。



## 查看系统信息

```shell
uname -a
```



## 后台运行程序

后台运行

```bash
nohup cmd & # 输出会重定向到nohup.out中或者nohup cmd >hello.log & 指定输出到hello.log中
```

```bash
yum install screen # 安装
screen -S [$Name] # 创建窗口
screen cmd # 在窗口中执行命令
screen -ls # 列出窗口
screen -r [$Name] # 再次连接ssh时恢复会话
```

查看后台程序

```shell
jobs -l # 列出
kill %num # 杀死后台作业
```



## 文件

### 创建文件

- `vi /vim filename`：文本编辑器

- `touch filename`：创建空文件

- 重定向创建

  ```shell
  echo "string" > filename
  cat > filename：输入文件内容，ctrl+d保存并退出
  ```

  



### 复制文件/目录

- `cp file_path xx`

  当xx为文件时，即创建file的副本xx

  当xx为目录时，即将file复制到xx目录下

  可选参数：-R  复制文件夹



### 文件链接

- 软链接/符号链接

  `ln -s 源文件名 快捷方式名`（软连接，只是创建了一个快捷方式）

- 硬链接

  `ln  源文件名 文件名`（硬链接，两个文件是同步的，就是对任意一个操作，都会影响到两个）  

- 二者区别

  删除了源文件，软连接就失效了，而硬链接相当于副本，不受影响。



### 重命名文件/目录

> 在Linux中，重命名文件称为移动（moving）

- `mv name1  name2`：更改文件名/目录名
- `mv name dir_path`：移动文件/目录
- `mv file_path   dir_path+file_name2`：移动文件并重命名



### 删除文件

> 在Linux中，删除（deleting）叫作移除（removing）。

- `rm  filename`
  - -i：删除前询问
  - -f：强制删除



### 读取文件

|   命令    |                作用                 |   可选参数   |
| :-------: | :---------------------------------: | :----------: |
|  cat/tac  |   从头到尾显示文件内容（tac相反）   | -n：显示行号 |
| head/tail |          读取前/后lines行           |  --lines=？  |
|    nl     |           显示行号和内容            |              |
|   sort    |         排序文件内容并显示          |              |
|   more    |          分页显示文件内容           |              |
|   uniq    |        去重后，显示文件内容         |              |
|   less    |       一次显示一屏的文件文本        |              |
|   paste   |  可以将两个文本按列拼接到一起显示   |              |
|  strings  | 读取二进制文件（cat不能读取二进制） |              |





### 搜索文件内容

`grep string filename` ：在file中查找匹配string的内容

参数：

|      |     功能     |                例子                |
| :--: | :----------: | :--------------------------------: |
|  -v  |   反向搜索   |          grep -v t file1           |
|  -n  |   显示行号   |          grep -n t file1           |
|  -e  | 多个匹配模式 | grep -e t -e f file1：匹配t和f字符 |
| 正则 |              |                                    |



### 查看文件类型/状态

```shell
file filename
stat filename
```



### 解压缩文件

赋予文件权限

`chmod +rw filename`

- tar

  ```shell
  解包：tar xvf FileName.tar
  打包：tar cvf FileName.tar DirName
  （注：tar是打包，不是压缩！）
  ```

  



### 文件权限

理解权限

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# ls -l
total 16
-rwxrwxr-x 1 rich rich 4882 2010-09-18 13:58 myprog
....

```

![image-20230225224057939](../images/image-20230225224057939.png)

-rwxrwxr-x解读为-，rwx，rwx，r-x

- 第一个字符

  | 字符 |      含义      |
  | :--: | :------------: |
  |  -   |    代表文件    |
  |  d   |    代表目录    |
  |  l   |    代表链接    |
  |  c   | 代表字符型设备 |
  |  b   |   代表块设备   |
  |  n   | n代表网络设备  |

- 后面9个字符，每三个为一组              

  | 字符 |        含义        | 每一组对应的安全级别 |
  | :--: | :----------------: | :------------------: |
  |  r   |  代表对象是可读的  |    文件属主的权限    |
  |  w   |  代表对象是可写的  |    属主成员的权限    |
  |  x   | 代表对象是可执行的 |    其他用户的权限    |

  > 若没有某种权限，在该权限位会出现单破折线。
  >
  > 属主成员指的是与文件属主在同一个用户组的成员
  >
  > 其他用户指的是与文件属主不在同一个用户组的成员



<img src="../images/image-20220927165623523.png" alt="image-20220927165623523" style="zoom:67%;" />



#### 改变权限

chmod

<img src="../images/image-20221030190551191.png" alt="image-20221030190551191" style="zoom: 67%;" />

八进制法

`sudo  chmod 777  filename`

> 777由上图可知是赋予全部权限
>



#### 设置默认权限

umask

![image-20230318143219603](../images/image-20230318143219603.png)

![image-20230318143232817](../images/image-20230318143232817.png)

文件权限=777-umask=755



## 目录

### 查看目录下的文件

- `ls`

| 可选参数 |               功能               |
| :------: | :------------------------------: |
|    -l    |           查看详细信息           |
|    -a    |   显示所有文件（包括隐藏文件）   |
|    -R    | 展示当前目录下的所有子目录和文件 |
|    -i    |        展示文件的索引节点        |
|    -F    |           展示文件类型           |

> -F
>
> 目录后加上`/`
>
> 可执行文件后加上`*`

![image-20220819201845329](../images/image-20220819201845329.png)

- `dir`
- `vdir` 等价于 `ls -l`



### 查看目录结构

`tree dir`

![image-20220818190348296](../images/image-20220818190348296.png)

### 创建目录

- `mkdir dir_name`：创建单个目录

- `mkdir dir1/dir2/dir3`：创建多个目录

  ![image-20220920223042434](../images/image-20220920223042434.png)



### 删除目录

- `rmdir dir_name`：删除空目录

- `rm  -rf  dir_name`：删除非空目录

  > 不使用-f参数的话，需要一一确定是否删除子目录和文件
  >
  > 更安全，但是更麻烦

  

### 切换到某个目录

`cd dir_path`



### 显示当前所在目录

`pwd`



## 进程--process

### 查看进程

- ps



![image-20220819201116476](../images/image-20220819201116476.png)

显示了程序的进程ID（Process ID，PID），它们运行在哪个终端（TTY），以及进程已用的CPU时间。

```bash
ps -ef
ps -aux | grep apache # 与grep联用查找apache进程
```

```bash
-A 显示所有进程
a 显示所有进程
-a 显示同一终端下所有进程
c 显示进程真实名称
e 显示环境变量
f 显示进程间的关系
r 显示当前终端运行的进程
-aux 显示所有包含其它使用的进程
```

- top

<img src="../images/image-20220922144049498.png" alt="image-20220922144049498" style="zoom: 67%;" />

> 与ps不同的点在于，ps只能显示一个时间点的进程信息，而top是实时更新，就像windows的任务管理器。
>

交互命令：输入q退出top命令。

- lsof

  ```shell
  lsof -p pid   //查看进程打开了哪些文件
  ```

  

### 结束进程

> 在Linux中，进程之间通过信号来通信。

<img src="../images/image-20220922144615297.png" alt="image-20220922144615297" style="zoom: 67%;" />

- `kill pid`：通过进程的pid来结束进程

- `kill -9 pid`：强制结束

- `killall  process_name`：通过进程名结束进程（支持通配符）

  ```shell
  killall http*
  上面的命令结束了所有以http开头的进程
  ```

  

  

  
  
  
  

## 用户



### 切换用户

su  -  用户名

高权限切换低权限无需密码，反之需要。



### 查看第一次登录到系统的用户

who am i





## 特殊符号

|      |                    功能                    |
| :--: | :----------------------------------------: |
|  #   |               注释后面的命令               |
|  ；  |                依次执行命令                |
|  \|  | 管道，前一个命令的输出作为后一个命令的输入 |
| \|\| |  如果前一条命令执行不成功则执行下一条命令  |
|  &   |                后台执行命令                |
|  &&  |   如果前一条命令执行成功则执行下一条命令   |
|  $?  |          存储上一条命令的执行结果          |







## 网络

### 查看ip

```shell
ip address show

ifconfig
```



### 抓取服务流量

```shell
sudo tcpdump -i lo port 6379 -w redis.pcapng
```





## 查找命令

![image-20221007162424354](../images/image-20221007162424354.png)

- `find`

  ![image-20230221171347974](../images/image-20230221171347974.png)

  ![image-20230324204821642](../images/image-20230324204821642.png)

  ```bash
  # 查找可写目录
  find / -type d -writable
  ```

- `locate`

  ```shell
  #更新索引数据库
  sudo updatedb
  ```

  

## 端口

> 端口是通过端口号来标记的，端口号只有整数，范围是从0~65535 

https://www.runoob.com/w3cnote/linux-check-port-usage.html

- lsof

  ```shell
  lsof -i:8000  //查看8000端口占用情况
  ```
  
- `netstat` 

  ```shell
  netstat -ntlp   //查看当前所有tcp端口
  netstat -ntulp | grep 80   //查看所有80端口使用情况
  netstat -ntulp | grep 3306   //查看所有3306端口使用情况
  ```

  ![image-20220804194025228](../images/image-20220804194025228.png)

  

## 服务

```shell
systemctl  command  服务
```

| command |       mean       |
| :-----: | :--------------: |
| status  |   查看服务状态   |
|  start  |     启动服务     |
| restart |     重启服务     |
|  stop   |     停止服务     |
| enable  |   开机自启服务   |
| disable | 取消开机自启服务 |

reload:不关闭unit的情况下,重新载入配置文件,让设置生效。
is- active:目前有没有正在运行中。
is- enabled:开机时有没有默认要启用这个unit。
kill:不要被kill这个名字吓着了,它其实是向运行unit的进程发送信号
show:列出unit的配置。
mask:禁用服务
unmask:取消对服务的禁用

list-unit-files|grep enable:列出所有自启动服务

![image-20221024194730467](../images/image-20221024194730467.png)

loaded：开机是否自启动，enabled说明是自启动的

active：现在是否启动，active（running)说明正在



## 使用技巧

![image-20230221171851141](../../../../blog/source/images/image-20230221171851141-1693403138821.png)

![image-20230221171905266](../../../../blog/source/images/image-20230221171905266-1693403138820.png)





### 为命令起别名

> `alias`命令是另一个shell的内建命令。命令别名允许你为常用的命令（及其参数）创建另一
>
> 个名称，从而将输入量减少到最低。



`alias 别名='原命令'`

下面为展示隐藏文件的ls命令起别名`la`

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# alias la='ls -a'
[root@izwz99bgx9y93dmv0mir0ez ~]# la
.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .cache  .cshrc  H1ve  .pip  .pki  .pydistutils.cfg  .ssh  .tcshrc  .viminfo  web
```

> 注意命令别名属于内部命令，一个别名仅在它所被定义的shell进程中才有效。
>
> 可以通过在~/.bashrc文件内编写，已达到永久生效的效果



### 执行历史命令

先用`history`命令查看历史命令记录，再用`！+ 命令编号`执行历史命令。

![image-20220923183323571](../../../../blog/source/images/image-20220923183323571-1693403159124.png)