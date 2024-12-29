---
title: Linux命令本质
date: 2023/5/31 19:48:25
categories:
- Linux
---

## 命令本质

命令实际上就是具有一定功能的二进制文件



## 命令位置

/bin,/usr/bin，默认都是全体用户使用

/sbin,/usr/sbin,默认root用户使用



## 执行过程

![img](../images/4dbf34717d434bb692683937feb52fea.png)



## 命令分类--外部和内建

> 内建命令实际上是shell程序的一部分，简单快速系统bash内置源码
> 比如：exit，history，cd，echo等。
>
> 外部命令是linux系统中的实用程序部分，外部命令的实体并不包含在shell中，但是其命令执行过程是由shell程序控制的。外部命令是在bash之外额外安装的，通常放在/bin，/usr/bin，/sbin，/usr/sbin等



- 查看外部命令存储位置

  ```bash
  echo $PATH
  ```

  ![image-20220710194138135](../images/image-20220710194138135-1686283886642.png)

  注：这里的PATH就是经常提到的环境变量

  > 当用户执行的是外部命令时，系统会在指定的多个路径中查找command的可执行文件，而定义这些路径的变量，就称为 PATH 环境变量，其作用就是告诉 Shell 待执行命令的可执行文件可能存放的位置，Shell 会在 PATH 变量包含的多个路径中逐个查找，直到找到为止（如果找不到，Shell 会提供用户“找不到此命令”）。

  

- 判断命令是否为外部还是内建命令

  ```bash
  type command
  ```

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# type cd
  cd is a shell builtin
  [root@izwz99bgx9y93dmv0mir0ez ~]# type vim
  vim is /usr/bin/vim
  [root@izwz99bgx9y93dmv0mir0ez ~]# type ps
  ps is hashed (/usr/bin/ps)
  ```

  

- 区别

  - 外部命令

    > 当外部命令执行时，会创建出一个子进程。这种操作被称为衍生（forking）。外部命令ps很方便显示出它的父进程以及自己所对应的衍生子进程。

    ```shell
    [root@izwz99bgx9y93dmv0mir0ez ~]# ps -f
    UID        PID  PPID  C STIME TTY          TIME CMD
    root      8986  8984  0 17:17 pts/0    00:00:00 -bash
    root      9054  8986  0 17:53 pts/0    00:00:00 ps -f
    ```

    ps进程的PPID为-bash的PID。

    ![image-20220710194138135](../images/image-20220923175907599.png)

    

    

  - 内建命令

    > 内建命令和外部命令的区别在于前者不需要使用子进程来执行。
    >
    > 因为既不需要通过衍生出子进程来执行，也不需要打开程序文件，内建命令的执行速度要更
    >
    > 快，效率也更高。

    

- 有些命令有多种实现方式

  ```shell
  [root@izwz99bgx9y93dmv0mir0ez ~]# type -a cd
  cd is a shell builtin
  cd is /usr/bin/cd
  ```

  > 对于有多种实现的命令，如果想要使用其外部命令实现，直接指明对应的文件就可以了。
  >
  > 例如，要使用外部命令pwd，可以输入/bin/pwd。

  







