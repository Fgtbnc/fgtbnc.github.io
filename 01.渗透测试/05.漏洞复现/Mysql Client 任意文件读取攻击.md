---
title: Mysql Client 任意文件读取攻击复现
date: 2023/5/31 19:47:25
categories:
- 漏洞复现
tags:
- 数据库攻防
---



# Mysql Client 任意文件读取攻击

## 脚本

https://github.com/allyshka/Rogue-MySql-Server

## 原理和攻击思路

[CSS-T | Mysql Client 任意文件读取攻击链拓展](https://paper.seebug.org/1112/)

简单的说就是

**正常的流程**

如果客户端要用load data local infile 将文件插入表中的话，客户端会先发一个请求包，这个请求包里包含了要插入的文件的路径。而服务器接下来返回一个Response TABULAR包，里面包含文件路径，然后客户端得到了许可才开始传输文件。

**漏洞产生点**

![img](../../../images/291ea879-e6dd-46a6-af48-4dd927485670.png-w331s)

> **服务端可以在任何查询语句后回复文件传输请求**

所以我们只需要伪造服务端返回的Response TABULAR包，并在该包中指定想要读取的文件，客户端就会把该文件的内容返回给服务端



## 条件

- 客户端必须启用LOCAL-INFILE 

  ```mysql
  连接时参数--local-infile=OFF 可以修复，或者更改全局变量local_infile(我在5.7下参数可以，更改全局变量不行？？)
  ```

- 客户端支持非SSL连接

  ```
  连接时参数--ssl-mode=VERIFY_IDENTITY 可以修复
  ```

- 目标web**存在连接数据库的功能**

  ```bash
  数据库弱口令扫描，连接检查
  网站重装漏洞（需要连接数据库）
  数据迁移服务
  Excle从数据库中同步数据到表格内
  ```

  

## 实验环境

```
客户端与服务端的mysql都是5.7版本
```

`mysql5.7`默认开启

![image-20230523163445312](../../../images/image-20230523163445312.png)

注：一开始使用服务端使用8版本不行



攻击者（服务端）起脚本

![image-20230523162503899](../../../images/image-20230523162503899.png)

![image-20230523162623757](../../../images/image-20230523162623757.png)

受害者连接

![image-20230523162534773](../../../images/image-20230523162534773.png)

攻击者查看`mysql.log`

![image-20230523162727437](../../../images/image-20230523162727437.png)

