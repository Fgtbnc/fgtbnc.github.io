---
title: 《Linux命令行与shell脚本编程大全》 笔记
date: 2023/5/31 19:48:25
categories:
- Linux
---



前言：

以下内容为阅读`Linux命令行与shell脚本编程大全.第3版 (布鲁姆，布雷斯纳汉) (z-lib.org)`笔记。

注：下面使用的linux系统为阿里云服务器Centos7（redhat）或者kali（debian）系统

------



# 了解shell

### shell类型

![image-20220818151112756](../images/image-20220818151112756-1686151304118.png)

> 当用户登录终端的时候，通常会启动一个默认的交互式shell。
>
> 系统究竟启动哪个shell，这取决于用户ID配置系统启动什么样的shell程序。
>
> 在/etc/passwd文件中，在用户ID记录的第7个字段中列出了默认的shell程序。



例如在kali中登录终端时使用的是zsh类型的shell.

```shell
cat /etc/passwd

khaz: x:1000:1000:khaz,,,:/home/khaz:/usr/bin/zsh
```



```shell
┌──(khaz㉿kali)-[~/桌面]
└─$ ls -F /usr/bin/zsh
/usr/bin/zsh*
```

说明/usr/bin/zsh是一个可执行程序



```shell
┌──(khaz㉿kali)-[~/桌面]
└─$ ls -l /bin/sh
lrwxrwxrwx 1 root root 4  3月 14  2022 /bin/sh -> dash
```

可以发现用户和系统使用的shell类型不同。



查看所有shell

```shell
khaz@DESKTOP-JCNAFF7:~$ cat /etc/shells
# /etc/shells: valid login shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/bin/dash
/usr/bin/dash
/usr/bin/tmux
/usr/bin/screen
/bin/zsh
/usr/bin/zsh
```

> 通常说的sh只是一个软连接，并不是真的有一个shell叫sh。
>
> 在debian系操作系统中，sh指向dash；
>
> 在centos系操作系统中，sh指向bash。



### shell的父子关系

![image-20220819202328859](../images/image-20220819202328859-1686151304119.png)

在右边的命令行中可以看到bash进程的ppid就是/user/bin/zsh的pid，所以二者是父子关系。

![](../images/image-20220819202343322-1696733797342.png)

多次创建bash

<img src="E:\typora img\image-20220819203036919.png" alt="image-20220819203036919" style="zoom: 67%;" />

<img src="E:\typora img\image-20220819203046906.png" alt="image-20220819203046906" style="zoom:67%;" />

退出当前shell，`exit`

<img src="E:\typora img\image-20220819203202199.png" alt="image-20220819203202199" style="zoom:67%;" />





启动shell时的参数

<img src="E:\typora img\image-20220923170946734.png" alt="image-20220923170946734" style="zoom:67%;" title=1/>

```shell
khaz@DESKTOP-JCNAFF7:~$ bash -r
khaz@DESKTOP-JCNAFF7:~$ cd ../
bash: cd: restricted
khaz@DESKTOP-JCNAFF7:~$
```



### 命令列表

> 以;隔开的多个命令

` pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls ; echo $BASH_SUBSHELL `

进程列表

> 创建一个子shell来执行命令列表

` (pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls ; echo $BASH_SUBSHELL)`

`echo $BASH_SUBSHELL`：输出子shell数量

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]#  (pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls ; echo $BASH_SUBSHELL)
/root
H1ve  web
/etc
/root
H1ve  web
1
```

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# ( pwd ; echo $BASH_SUBSHELL)
/root
1
[root@izwz99bgx9y93dmv0mir0ez ~]# ( pwd ;(echo $BASH_SUBSHELL))
/root
2
```

可以看出有几个括号就有几个子shell



# 使用linux环境变量

- 定义

  > **存储有关shell会话和工作环境的信息的变量称为环境变量**

- 分类

  - 全局变量

  - 局部变量

    > 这里的范围指的是变量存在于哪些shell会话中

  

### 全局环境变量

> 系统环境变量基本上都是使用全大写字母，以区别于普通用户的环境变量。

查看全局变量

- `env` 或者`printenv`：**只**输出全部的**全局**变量

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# env  (printenv输出的与其相同)
  XDG_SESSION_ID=897
  HOSTNAME=izwz99bgx9y93dmv0mir0ez
  TERM=xterm
  SHELL=/bin/bash
  HISTSIZE=1000
  SSH_TTY=/dev/pts/0
  USER=root
  ....
  ```

- `printenv  var_name`或者 `echo $var_name`：输出特定变量的值。

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# printenv USER
  root
  [root@izwz99bgx9y93dmv0mir0ez ~]# echo $USER
  root
  
  ```




全局环境变量存在于所有的shell会话中

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# bash
[root@izwz99bgx9y93dmv0mir0ez ~]# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root     11337 11335  0 20:14 pts/0    00:00:00 -bash
root     11371 11337  0 20:31 pts/0    00:00:00 bash
root     11382 11371  0 20:31 pts/0    00:00:00 ps -f
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $HOME
/root
[root@izwz99bgx9y93dmv0mir0ez ~]# exit
exit
[root@izwz99bgx9y93dmv0mir0ez ~]# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root     11337 11335  0 20:14 pts/0    00:00:00 -bash
root     11383 11337  0 20:32 pts/0    00:00:00 ps -f
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $HOME
/root

```

> 在这个例子中，用bash命令生成一个子shell后，显示了HOME环境变量的当前值，这个值和父shell中的一模一样，都是/root。





### 局部环境变量

查看局部变量

> 在Linux系统并没有一个只显示局部环境变量的命令。

- `set`

  > set命令会显示为某个特定进程设置的所有环境变量，包括局部变量、全局变量以及用户定义变量。

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# set
  BASH=/bin/bash
  BASHOPTS=checkwinsize:cmdhist:expand_aliases:extquote:force_fignore:histappend:hostcomplete:interactive_comments:login_shell:progcomp:promptvars:sourcepath
  BASH_ALIASES=()
  BASH_ARGC=()
  BASH_ARGV=()
  BASH_CMDS=()
  BASH_LINENO=()
  BASH_SOURCE=()
  BASH_VERSINFO=([0]="4" [1]="2" [2]="46" [3]="2" [4]="release" [5]="x86_64-redhat-linux-gnu")
  BASH_VERSION='4.2.46(2)-release'
  COLUMNS=210
  DIRSTACK=()
  EUID=0
  GROUPS=()
  ....
  ```

  > set命令会显示出全局变量、局部变量以及用户定义变量。它还会按照字母顺序对结果进行排序。
  >
  > env和printenv命令同set命令的区别在于前两个命令不会对变量排序，也不会输出局部变量和用户定义变量。

  




### 设置局部用户自定义变量

> 一旦启动了bash shell（或者执行一个shell脚本），就能创建在这个shell进程内可见的局部变量了。
>
> 可以通过等号给环境变量赋值，值可以是数值或字符串。

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $first_var

注：因为没有定义first_var变量，所以输出为空。

[root@izwz99bgx9y93dmv0mir0ez ~]# $first_var=hello world
-bash: =hello: command not found
注：定义变量时无需加上$

[root@izwz99bgx9y93dmv0mir0ez ~]# first_var=hello world
-bash: world: command not found
注：在为变量赋值字符串时，如果字符串有引号，就需要用单引号括起来。
引用：没有单引号的话，bash shell会以为下一个词是另一个要执行的命令

root@izwz99bgx9y93dmv0mir0ez ~]# first_var ='hello world'
-bash: first_var: command not found
[root@izwz99bgx9y93dmv0mir0ez ~]# first_var= 'hello world'
-bash: hello world: command not found
注：变量和=和值之间不能有空格，否则shell会把其当成命令。


[root@izwz99bgx9y93dmv0mir0ez ~]# first_var='hello world'
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $first_var
hello world

```



局部变量的范围

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root     11337 11335  0 20:14 pts/0    00:00:00 -bash
root     11415 11337  0 20:57 pts/0    00:00:00 ps -f
[root@izwz99bgx9y93dmv0mir0ez ~]# bash
[root@izwz99bgx9y93dmv0mir0ez ~]# child_var='hello khaz'
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $child_var
hello khaz
[root@izwz99bgx9y93dmv0mir0ez ~]# exit
exit
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $child_var


```



### 设置全局变量

> 创建全局环境变量的方法是先**创建**一个局部环境变量，然后再把它**导出**到全局环境中。

`export var_name`：导出子shell的局部变量到全局环境中（**临时修改**）

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# ps --forest
  PID TTY          TIME CMD
11566 pts/1    00:00:00 bash
11585 pts/1    00:00:00  \_ ps
[root@izwz99bgx9y93dmv0mir0ez ~]# a='khaz'
[root@izwz99bgx9y93dmv0mir0ez ~]# bash
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $a

[root@izwz99bgx9y93dmv0mir0ez ~]# exit
exit
[root@izwz99bgx9y93dmv0mir0ez ~]# export a
[root@izwz99bgx9y93dmv0mir0ez ~]# bash
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $a
khaz

```

> 我发现只能在登录终端的那个shell中export，不能在子shell中export？
>
> 因为export的作用是让子shell可以继承父shell导出过的变量。
>
> 并且如果在子shell中修改全局变量的值，也不会影响到父shell中该全局变量的值。





### 删除环境变量

`unset var_name`：删除环境变量

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# a=1
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $a
1
[root@izwz99bgx9y93dmv0mir0ez ~]# unset a
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $a
```





### 设置PATH环境变量

> PATH环境变量定义了用于进行命令和程序查找的目录。

下面演示了创建自定义命令，并可以在任何路径下使用该命令。

- 创建自定义命令，命令内容为输出'hello khaz'，并将其移动到存放自定义命令的目录下

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# vim hello 
  #!/bin/bash
  echo 'hello khaz'
  :wq
  [root@izwz99bgx9y93dmv0mir0ez ~]# mkdir my_cmd
  [root@izwz99bgx9y93dmv0mir0ez ~]# mv hello my_cmd/
  ```

- 因为与命令不在同一路径下，提示找不到命令

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# hello
  -bash: hello: command not found
  ```

- 查看并增加PATH（路径以：分隔）

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# echo $PATH
  /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
  [root@izwz99bgx9y93dmv0mir0ez ~]# PATH=$PATH:/root/my_cmd/
  [root@izwz99bgx9y93dmv0mir0ez ~]# echo $PATH
  /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/root/my_cmd/
  ```

- 再次输入命令成功执行

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# hello
  hello khaz
  ```

  > 这种方法对PATH变量的修改只能持续到退出或重启系统。





### 让环境变量的作用持久化

> 把我们设置的环境变量放在shell的启动文件/环境文件中。
>
> 或者开机启动项`/etc/init.d/`和`/etc/profile.d/`

- 启动文件/环境文件

  > 登入Linux系统启动一个bash shell时，默认情况下bash会在几个文件中查找命令。
  >
  > 这些文件叫作启动文件或环境文件。

- 5个不同的启动文件

   /etc/profile 

   $HOME/.bash_profile 

   $HOME/.bashrc 

   $HOME/.bash_login 

   $HOME/.profile 

- /etc/profile文件

  > /etc/profile文件是bash shell默认的的主启动文件。
  >
  > 只要你登录了Linux系统，bash就会执行/etc/profile启动文件中的命令。

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# cat /etc/profile
  .....
  .....
  for i in /etc/profile.d/*.sh ; do
      if [ -r "$i" ]; then
          if [ "${-#*i}" != "$-" ]; then 
              . "$i"
          else
              . "$i" >/dev/null
          fi
      fi
  done
  ....
  ....
  ```

  > 其中的for语句为Linux系统提供了一个放置特定应用程序启动文件的地方，当用户登录时，shell会执行这些文件。

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# ls -al /etc/profile.d/
  total 64
  drwxr-xr-x.  2 root root 4096 Oct 15  2017 .
  drwxr-xr-x. 82 root root 4096 Sep 24 23:07 ..
  -rw-r--r--.  1 root root  123 Jul 31  2015 less.csh
  -rw-r--r--.  1 root root  121 Jul 31  2015 less.sh
  -rw-r--r--   1 root root  105 Aug  2  2017 vim.csh
  -rw-r--r--   1 root root  269 Aug  2  2017 vim.sh
  -rw-r--r--.  1 root root  164 Jan 28  2014 which2.csh
  -rw-r--r--.  1 root root  169 Jan 28  2014 which2.sh
  -rw-r--r--. 1 root root 1741 Feb 20 05:44 lang.csh 
  -rw-r--r--. 1 root root 2706 Feb 20 05:44 lang.sh
  .....
  ```

  > 不难发现，有些文件与系统中的特定应用有关。
  >
  > 大部分应用都会创建两个启动文件：一个供bash shell使用（使用.sh扩展名），一个供c shell使用（使用.csh扩展名）。
  >
  > 其中lang.csh和lang.sh文件是用来设置字符集的。

- $HOME目录下的启动文件

  > 这些启动文件都起着同一个作用：提供一个用户专属的启动文件来定义该用户所用到的环
  >
  > 境变量。

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# ls -a $HOME
  .  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .cache  cqhttp  .cshrc  H1ve  my_cmd  .pip  .pki  .pydistutils.cfg  .ssh  .tcshrc  .viminfo  .viminfo.tmp  web
  
  ```

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# cat $HOME/.bash_profile
  # .bash_profile
  
  # Get the aliases and functions
  if [ -f ~/.bashrc ]; then
  	. ~/.bashrc
  fi
  
  # User specific environment and startup programs
  
  PATH=$PATH:$HOME/bin
  
  export PATH
  
  ```

  - 这里先会判断.bashrc文件是否存在，若存在先执行这个文件。

    ```shell
    [root@izwz99bgx9y93dmv0mir0ez ~]# cat .bashrc 
    # .bashrc
    
    # User specific aliases and functions
    
    alias rm='rm -i'
    alias cp='cp -i'
    alias mv='mv -i'
    
    # Source global definitions
    if [ -f /etc/bashrc ]; then
    	. /etc/bashrc
    
    ```

    这里可以设置别名，比如我添加`alias la='ls -a'`,重启shell后

    ```shell
    [root@izwz99bgx9y93dmv0mir0ez ~]# cat ~/.bashrc 
    # .bashrc
    
    # User specific aliases and functions
    
    alias rm='rm -i'
    alias cp='cp -i'
    alias mv='mv -i'
    alias la='ls -a'
    # Source global definitions
    if [ -f /etc/bashrc ]; then
    	. /etc/bashrc
    fi
    [root@izwz99bgx9y93dmv0mir0ez ~]# la
    .  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .cache  cqhttp  .cshrc  H1ve  my_cmd  .pip  .pki  .pydistutils.cfg  .ssh  .tcshrc  .viminfo  .viminfo.tmp  web
    
    ```

  - `PATH=$PATH:$HOME/bin`则应该是登录shell后自动添加路径到PATH中

    ```shell
    编辑为PATH=$PATH:$HOME/bin:$HOME/my_cmd
    重启shell后/source立即生效后
    
    [root@izwz99bgx9y93dmv0mir0ez ~]# echo $PATH
    /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/root/my_cmd
    [root@izwz99bgx9y93dmv0mir0ez ~]# hello
    hello khaz
    
    ```

    

    


### 环境变量立即生效

```shell
source xxx
```



### 环境变量特性

> 环境变量有一个很酷的特性就是，它们可作为数组使用。

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# arrary=(1 2 3)
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $arrary
1
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $arrary[1]
1[1]
[root@izwz99bgx9y93dmv0mir0ez ~]# echo ${arrary[1]}
2
```

在使用变量时最好将变量名用{}括起来，否则可能会解析错误。





# shell脚本编写

1. 创建shell脚本

   > 在创建shell脚本文件时，必须在文件的第一行指定要使用的shell。其格式为：`#!/bin/bash `
   >
   > 在通常的shell脚本中，井号（#）用作注释行。shell并不会处理shell脚本中的注释行。然而，
   >
   > shell脚本文件的第一行是个例外，#后面的惊叹号会告诉shell用哪个shell来运行脚本。

   ```shell
   #!/bin/bash
   echo hello world
   ```

2. 设置权限

   `sudo chmod +x file_name `

3. 运行脚本

   - shell 脚本文件名
   - chmod +x 脚本文件名; 脚本文件名
   - . 脚本文件名
   - source 脚本文件名

脚本文件名如`./name.sh`

<img src="E:\typora img\20210321121053755.png" alt="在这里插入图片描述" style="zoom: 80%;" />

### 变量

#### 定义与使用

> 如果要用到变量，使用$；
>
> 如果要操作变量，不使用$。
>
> 这条规则的一个例外就是使用printenv显示某个变量的值。

用到变量 → 用到变量的值

```shell
echo $a  #输出变量的值
printenv PATH #用到变量的值，但不需要加$
```

操作变量

```shell
a=1 #定义变量
export a #导出变量
unset a #删除变量
```



#### 特性

能够让变量作为命令行参数

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $HOME
/root
[root@izwz99bgx9y93dmv0mir0ez ~]# ls $HOME
H1ve  web
[root@izwz99bgx9y93dmv0mir0ez ~]# ls /root
H1ve  web
```



内敛执行

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# cat `ls`
[root@izwz99bgx9y93dmv0mir0ez ~]# cat $(ls)
>>读取当前目录的所有文件
将ls的输出作为cat的输入进行执行。
```

> 先执行的命令是由当前shell创建的子shell执行的。



加上{}用于区分变量

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# a=b
[root@izwz99bgx9y93dmv0mir0ez ~]# echo $ab
 	
#因为找不到变量ab，所以输出为空
[root@izwz99bgx9y93dmv0mir0ez ~]# echo ${a}b
bb

```





### 字符串

当变量值含有空格时，就要加上引号。

```shell
[root@izwz99bgx9y93dmv0mir0ez ~]# b= world
-bash: world: command not found
```

单引号：不能解析变量，只会原样输出

双引号：能够解析变量

- 获取字符串长度   `${#string}`

  ```shell
  ┌──(khaz?kali)-[~]
  └─$ str='hello khaz'
                                                                    
  ┌──(khaz?kali)-[~]
  └─$ echo ${#str}                                                                  
  10
  
  ```

- 切片  `${string:num1:num2}`

  > num1可以省略，默认为0，num2必须有值，支持负数

  ```shell
  ┌──(khaz㉿kali)-[~]
  └─$ str='hello khaz'
  
  ┌──(khaz㉿kali)-[~]
  └─$ echo ${str::2} 
  he
  
  ┌──(khaz㉿kali)-[~]
  └─$ echo ${str::-1}
  hello kha
  ```

- 截取

  ```shell
  str="www.runoob.com/linux/linux-shell-variable.html"
  echo "str    : ${str}"
  echo "str#*/    : ${str#*/}"   # 从 字符串开头 删除到 左数第一个'/'
  echo "str##*/    : ${str##*/}"  # 从 字符串开头 删除到 左数最后一个'/'
  echo "str%/*    : ${str%/*}"   # 从 字符串末尾 删除到 右数第一个'/'
  echo "str%%/*    : ${str%%/*}"  # 从 字符串末尾 删除到 右数最后一个'/'
  >>
  str     : www.runoob.com/linux/linux-shell-variable.html
  str#*/  : linux/linux-shell-variable.html
  str##*/ : linux-shell-variable.html
  str%/*  : www.runoob.com/linux
  str%%/* : www.runoob.com
  ```

- 替换

  单个

  ```shell
  string="text, dummy, text, dummy"
  echo ${string/text/TEXT}
  TEXT, dummy, text, dummy
  ```

  全部

  ```shell
  string="text, dummy, text, dummy"
  echo ${string//text/TEXT}
  TEXT, dummy, TEXT, dummy
  ```









### 脚本参数

<img src="E:\typora img\image-20221005173522875.png" alt="image-20221005173522875" style="zoom: 67%;" />

```shell
#!/bin/bash

echo "Shell 传递参数实例！";
echo '$0' :"执行的文件名 $0";
echo '$1' :"第一个参数为 $1";
echo '$2' :"第二个参数为 $2";
echo '$3' :"第三个参数为 $3";
echo "传递到脚本的参数个数为 $#";
echo '$*' "：$*";
echo '$@' "：$@";
echo '$$' :"脚本进程pid $$";
echo "ps :`ps`";
echo '$!' :"后台最后一个进程的pid $!";
echo '$-' :"显示shell参数 $-";
echo '$?' :"显示最后一个命令的退出状态 $?";
```

```shell
┌──(root㉿kali)-[/home/khaz]
└─# ./shell.sh 1 2 3
Shell 传递参数实例！
$0 :执行的文件名 ./shell.sh
$1 :第一个参数为 1
$2 :第二个参数为 2
$3 :第三个参数为 3
传递到脚本的参数个数为 3
$* ：1 2 3
$@ ：1 2 3
$$ :脚本进程pid 2702
ps :    PID TTY          TIME CMD
   2276 pts/0    00:00:00 su
   2277 pts/0    00:00:02 zsh
   2702 pts/0    00:00:00 shell.sh
   2703 pts/0    00:00:00 ps
$! :后台最后一个进程的pid 
$- :显示shell参数 hB
$? :显示最后一个命令的退出状态 0

```







### 重定向

https://www.k0rz3n.com/2018/08/05/Linux%E5%8F%8D%E5%BC%B9shell%EF%BC%88%E4%B8%80%EF%BC%89%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6%E4%B8%8E%E9%87%8D%E5%AE%9A%E5%90%91/

> 当Linux启动的时候会默认打开三个文件描述符，分别是：
>
> 标准输入standard input 0 （默认设备键盘）
> 标准输出standard output 1（默认设备显示器）
> 错误输出：error output 2（默认设备显示器）

<img src="E:\typora img\重定向1.png" alt="重定向1.png" style="zoom: 50%;" />

> **注意：**
>
> （1）以后再打开文件，描述符可以依次增加
> （2）一条shell命令，都会继承其父进程的文件描述符，因此所有的shell命令，都会默认有三个文件描述符。
>
> **文件所有输入输出都是由该进程所有打开的文件描述符控制的。（Linux一切皆文件，就连键盘显示器设备都是文件，因此他们的输入输出也是由文件描述符控制）**
>
> **重定向就是针对文件描述符的操作**



重定向符号个数

1个：写入并覆盖  

2个：追加



#### 输出重定向

将命令结果输出到文件中，即将输出文件符1/2指向文件，默认为1。

- 标准输出重定向   > file

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez study]# ls > file
  [root@izwz99bgx9y93dmv0mir0ez study]# cat file
  errorlog
  file
  rightlog
  ```

  <img src="E:\typora img\重定向7.png" alt="重定向7.png" style="zoom:50%;" />

- 标准错误输出重定向   2> file

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez study]# touch errorlog
  [root@izwz99bgx9y93dmv0mir0ez study]# afaaafa 2> errorlog
  [root@izwz99bgx9y93dmv0mir0ez study]# cat errorlog
  -bash: afaaafa: command not found
  
  ```

  <img src="E:\typora img\重定向7-1664726910690.png" alt="重定向7" style="zoom:50%;" />

- 标准错误输出和标准输出重定向  &> file ，>& file， >file 2>&1

  > 此处**&>**或者**>&**视作整体，二者是一样的，分开没有单独的含义。
  >
  > & 目的是为了区分数字名字的文件和文件描述符，如果没有& 系统会认为是将文件描述符重定向到了一个数字作为文件名的文件，而不是一个文件描述符

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez study]# ls &> file
  [root@izwz99bgx9y93dmv0mir0ez study]# cat file
  errorlog
  file
  rightlog
  [root@izwz99bgx9y93dmv0mir0ez study]# dasdasd &>> file
  [root@izwz99bgx9y93dmv0mir0ez study]# cat file
  errorlog
  file
  rightlog
  -bash: dasdasd: command not found
  
  ```

  <img src="E:\typora img\重定向9.png" alt="重定向9.png" style="zoom:50%;" />

##### 注意点

顺序不能更改

`cmd > file 2>&1` →  ``cmd 2>&1 > file `

结果是标准输出和标准错误输出不是指向同一个文件。

<img src="E:\typora img\重定向11.png" alt="重定向11.png" style="zoom:50%;" />



#### 输入重定向

- 标准输入重定向

  将文件内容重定向到命令中，即将标准输入文件描述符0指向文件。

  - < file

    ```shell
    [root@izwz99bgx9y93dmv0mir0ez ~]# cat file
    /root
    [root@izwz99bgx9y93dmv0mir0ez ~]# ls < file
    1  cqhttp  H1ve  my_cmd  output  test  usr  web  YesBot_ws_Go_CQHTTP-main
    
    ```

    <img src="E:\typora img\重定向3.png" alt="重定向4.png" style="zoom: 50%;" />

    

  - 内联输入重定向  << file

    > 这种方法无需使用文件进行重定向，只需要在命令行中指定用于输入重定向的数据就可以了。

    ```shell
    [root@izwz99bgx9y93dmv0mir0ez ~]# cat << flag                                           
    heredoc> a
    heredoc> b
    heredoc> c
    heredoc> flag
    a
    b
    c
    ```

    > 指定一个文本标记（任何字符串都可作为文本标记，我这里是flag），来划分输入数据的开始和结尾。

>所以cat的输出为a,b,c

  

  

#### /dev/null文件

> /dev/null 是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 /dev/null 文件非常有用，将命令的输出重定向到它，会起到"禁止输出"的效果。





### 运算符

数学运算 `expr`

```shell
┌──(khaz㉿kali)-[~]
└─$ a=2 b=1 

┌──(khaz㉿kali)-[~]
└─$ echo `expr $a+$b`
2+1

┌──(khaz㉿kali)-[~]
└─$ echo `expr $a + $b`
3

```

注意：变量和运算符之间要用空格隔开才可以。运算表达式要用反引号包围。







![image-20221005183150173](../images/image-20221005183150173-1686151304119.png)





### IF语句

条件表达式

最好用[[   ]]，并注意间隔

```shell
if [[ condition1 ]]  
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```

```shell
if condition1; then command1; elif condition2; then command2; else commandN;fi;
```

需要注意的是在shell中0表示True，其他的表示False（命令执行状态）

```shell
┌──(khaz㉿kali)-[~/桌面]
└─$ if [[ 0 ]]
then
echo 0 is true
fi
>>0 is true

```

> 在bash和unix shell中，返回值通常不是布尔值。它们是整数退出代码，用来判断代码是否执行成功。
>
> 如果`0`是错误的，那么程序只能显示一种错误。
>
> 如果`0`是正确的，那么任何非零值都是错误的，我们就可以使用1-255之间的任何数字来表示错误。这意味着我们可以有许多不同类型的错误。
>
> 例如`1`是一个一般性错误，`126`表示文件无法执行，`127`表示“找不到命令”，等等。

```shell
┌──(khaz㉿kali)-[~/桌面]
└─$ ls
1.sh  2.sh  ctf  fpm.py  sstilabs-master  test.sh
                                                                                                 
┌──(khaz㉿kali)-[~/桌面]
└─$ echo $!    #$!保存最后一条命令执行的退出状态码
0

```



### FOR语句

bash下

```shell
for ((i=1;i<5;i++));
do
	echo $i   
done                                                                                        
```

```shell
#!/bin/bash
echo "-- \$* 演示 ---"
for i in "$*"; do
    echo $i
done

echo "-- \$@ 演示 ---"
for i in "$@"; do
    echo $i
done
```

![image-20221113230122214](../images/image-20221113230122214-1686151304119.png)



### while语句

```shell
#!/bin/bash
count=1
while [ "$count" -le 10 ] 
do
        echo $count
        count=$(($count+1))
done
```

### shell统计当前文件夹下的文件个数、目录个数

https://blog.csdn.net/weixin_44203158/article/details/109727981

#### sort

https://blog.csdn.net/linggang_123/article/details/118552536



#### grep

##### 在/etc/目录下寻找文件里包含mysql字符的文件，并打印其文件名。

```shell
grep -rl 'mysql' /etc
```

-r：递归查找

-l：只打印文件名





#### sed

s模式---字符替换

```shell
┌──(khaz㉿kali)-[~/桌面]
└─$ cat data        
The quick brown fox jumps over the lazy dog. 
The quick brown fox jumps over the lazy dog. 
The quick brown fox jumps over the lazy dog. 
The quick brown fox jumps over the lazy dog.
                                                                                                 
┌──(khaz㉿kali)-[~/桌面]
└─$ sed 's/dog/cat/' data  
The quick brown fox jumps over the lazy cat. 
The quick brown fox jumps over the lazy cat. 
The quick brown fox jumps over the lazy cat. 
The quick brown fox jumps over the lazy cat.

┌──(khaz㉿kali)-[~/桌面]
└─$ cat data
The quick brown fox jumps over the lazy dog. 
The quick brown fox jumps over the lazy dog. 
The quick brown fox jumps over the lazy dog. 
The quick brown fox jumps over the lazy dog.
```

> sed编辑器并不会修改文本文件的数据。它只会将修改后的数据发送到STDOUT。如果你查看原来的文本文件，它仍然保留着原始数据。
>
> 将sed结果重定向到其他文件即可。

参数 -e

多条sed命令执行

` sed -e 's/brown/green/; s/dog/cat/' data1.txt` 



参数 -f

`sed -f script1.sed data1.txt`  实现了` sed -e 's/brown/green/; s/dog/cat/' data1.txt`

script1.sed

```shell
s/brown/green/
s/dog/cat/
```

-n 不产生输出



#### awk

这个命令主要的作用就相当于split，format，字符串分割，格式化

输出所有用户

```
cat /etc/passwd | awk -F ':' '{print $1}'
```

> awk -F指定域分隔符为':'，将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域。
>
> 默认分隔符为空格





## $IFS

类似于python的.split()方法

$IFS内部域分隔符,默认为空格/Tab/换行

也可以指定IFS的值，将值替换为空格/Tab/换行

```shell
string="k?haz"
IFS='?'
echo $string
>>k haz
```


