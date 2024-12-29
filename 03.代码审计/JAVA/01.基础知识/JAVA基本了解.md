---
title: Java泛型
date: 2023/5/1 15:00:25
categories:
- Java
---

#  JAVA基本了解

## 版本问题

[(71条消息) Java--Java版本和JDK版本_java版本和jdk版本区别_MinggeQingchun的博客-CSDN博客](https://blog.csdn.net/MinggeQingchun/article/details/120578602)

![image-20230501122332032](E:\typora img\image-20230501122332032.png)

JDK8u311 → JAVA8

![image-20230425225221871](E:\typora img\image-20230425225221871.png)

![image-20230425225139403](E:\typora img\image-20230425225139403.png)

![image-20230425225121609](E:\typora img\image-20230425225121609.png)

jar包实际上就是一个zip格式的压缩文件



## 运行java程序

Java是将代码编译成一种“字节码”，它类似于抽象的CPU指令，然后，针对不同平台编写虚拟机，不同平台的虚拟机负责加载字节码并执行，这样就实现了“一次编写，到处运行”的效果。

![image-20230425225313657](E:\typora img\image-20230425225313657.png)

**一个Java源码只能定义一个`public`类型的class，并且class名称和文件名要完全一致；**

> 如文件名为demo.java，代码中定义的`public`类即为`public class demo`

注意：

我们通常说的Java 8，Java 11，Java 17，是指JDK的版本，也就是JVM的版本

而每个版本的JVM，它能执行的class文件版本也不同。例如，Java 11对应的class文件版本是55，而Java 17对应的class文件版本是61。

**高版本的JDK可编译输出低版本兼容的class文件**，但需注意，低版本的JDK可能不存在高版本JDK添加的类和方法，导致运行时报错。

运行时使用哪个JDK版本，编译时就尽量使用同一版本编译源码。



## 基本语法

### 数组

```java
int[] arr1 = new int[4];//这⾥的4代表的是数组的空间，就当作是存放元素的个数
int[] arr2 = {1,2,3,4};
int[] arr3 = new int[]{1,2,3,4};
```

![image-20231029195008386](../images/image-20231029195008386.png)



### 类与对象

#### 概念

![image-20230425225605486](E:\typora img\image-20230425225605486.png)

![image-20230425225757719](E:\typora img\image-20230425225757719.png)

#### 封装

**定义**

> 将类的某些信息隐藏在类的内部，不允许外部程序直接访问，⽽是通过该类的提供的⽅法来实现对隐藏信息的操作和访问

**如何实现**

定义private字段和pubilc方法来操作private字段

> 这样外部代码不能直接修改`private`字段。
>
> 但是，外部代码可以调用方法`setName()`和`setAge()`来间接修改`private`字段。
>
> 然后在`set`方法内部，我们可以编写检查代码，如果传入的值不能通过检查就会报错，保证了外部代码就没有任何机会把`age`设置成不合理的值。

定义private方法

> `private`方法，**外部代码无法调用**，但是，**内部方法可以调用它**。
>
> 方法可以封装一个类的对外接口，调用方不需要知道也不关心字段是否存在



#### Java Bean

> JavaBean是一种符合命名规范的`class`，它通过`getter`和`setter`来定义属性；

例子

```java
public class Person {
    private String name;
    private int age;

    public String getName() { return this.name; }
    public void setName(String name) { this.name = name; }

    public int getAge() { return this.age; }
    public void setAge(int age) { this.age = age; }
}
```



#### 继承

**特性总结**

- Java中只⽀持单继承【不能同时继承两个类】 
- ⽀持多层继承（继承体）【例⼦：demoB继承类demoA，demoC继承了denoB，相当于也继承了demoA】 
- ⼦类可以使⽤⽗类，⽗类不允许使⽤⼦类
- 只能继承public和protected修饰的属性和⽅法（不管⼦类⽗类是否在同⼀个包⾥） 
- ⼦类不能继承⽗类的构造⽅法和私有属性
-  ⼦类可以拥有⾃⼰的属性和⽅法，即⼦类可以对⽗类进⾏扩展
- ⼦类中所有的构造⽅法都会默认去访问⽗类的构造⽅法



#### 多态

- 要有继承
- 要有重写
- ⽗类引⽤指向⼦类对象

```java
public class demo1 { 
    public static void main(String[] args) 
    { 
        Father f = new Son();//⽗类引⽤指向⼦类对象 
        f.eat();//指向⼦类 
        } 
    }
class Father{ 
	public void eat(){ 
		System.out.println("吃饭"); 
	} 
} 
class Son extends Father{
	public void eat(){ 
	System.out.println("吃零⾷"); 
	} 
}
```



#### 抽象类

> 如果一个`class`定义了方法，但没有具体执行代码，这个方法就是抽象方法，抽象方法用`abstract`修饰。
>
> 因为无法执行抽象方法，因此这个类也必须申明为抽象类（abstract class）。

```java
abstract class Person {
    public abstract void run();
}
```

> 使用`abstract`修饰的类就是抽象类。我们无法实例化一个抽象类：

```java
Person p = new Person(); // 编译错误
```

> 无法实例化的抽象类有什么用？
>
> 因为抽象类本身被设计成只能用于被继承，因此，抽象类可以强迫子类实现其定义的抽象方法，否则编译会报错。因此，抽象方法实际上相当于定义了“规范”：即规定高层类的接口，从而保证所有子类都有相同的接口实现，这样，多态就能发挥出威力。



#### 接口

为什么有接口

> Java语⾔的继承是单⼀继承，⼀个⼦类只能有⼀个⽗类。那么接⼝，就是Java为我们提供的⼀种机制，来解决继承单⼀的局限性。
>
> 举一个例子
>
> 以防盗⻔为例，锁可以开锁和上锁，将⻔和锁定义为抽象类，防盗⻔可以继承⻔的同时⼜继承锁吗？
>
> - 将⻔定义为抽象类 
> - 锁定义为接⼝ 
> - 防盗⻔继承⻔并实现锁的接⼝
>
> 这样防盗⻔可以继承⻔的同时⼜继承锁

Demo

> 如果一个抽象类没有字段，所有方法全部都是抽象方法，就可以把该抽象类改写为接口：`interface`。

```java
interface Person {
    void run();
    String getName();
}
```

当一个具体的`class`去实现一个`interface`时，需要使用`implements`关键字。

```java
class Student implements Person {
    private String name;

    public Student(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println(this.name + " run");
    }

    @Override
    public String getName() {
        return this.name;
    }
}
```

接口的继承

![image-20230425233728359](E:\typora img\image-20230425233728359.png)



#### 静态字段和静态方法

> 修饰符static
>
> 静态方法和静态字段属于`class`而不属于实例

![image-20230425234023210](E:\typora img\image-20230425234023210.png)

使用类名来访问

```java
Person.setNumber(99);
System.out.println(Person.number);
```



#### 包

![image-20231030125226540](../images/image-20231030125226540.png)

![image-20231030125257641](../images/image-20231030125257641.png)

> 一个包就是一个命名空间
>
> 真正的完整类名是`包名.类名`

```java
package xxx; // 申明包名

import packge_name.class_name; // 导入其他包中的类
```

#### 常用类

![img](../images/java.lang.png)

##### String对象

> String不是⼀个数据类型，⽽是引⽤数据类型，属于Java提供的⼀个类
>
> Java中的String是不可变的，因为String类的存储是通过final修饰的char[]数组来存放结果的，不可更改。

```java
public class Test {
    public static void main(String args[]){
        String s = "Hello World";
        System.out.println(s.hashCode());
        System.out.println(s.toUpperCase().hashCode());
        System.out.println(s);
    }
}
```

![image-20230428115645212](E:\typora img\image-20230428115645212.png)

##### IO

内存数据与外存数据，使用字节流或字符流

```java
import java.io.*;

public class Test {
    public static void main(String[] args)  throws IOException{
        // getInputStream()获得结果
        InputStream inputStream = Runtime.getRuntime().exec("whoami").getInputStream();
        // 用来缓存命令执行结果数据
        byte[] cache = new byte[1024];
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        int readLen = 0;
        // read方法将数据读取到cache字节数组中，如果返回-1说明到了结尾，数据读取完毕
        while ((readLen = inputStream.read(cache))!=-1){
            // 数据读取完毕后，write方法将cache数组中从0到readlen的数据写入输出流中
            byteArrayOutputStream.write(cache, 0, readLen);
        }
        System.out.println(byteArrayOutputStream);
    }
}
```



### 异常

**分类**

![image-20231029200926591](../images/image-20231029200926591.png)

**异常处理**

![image-20231029200959809](../images/image-20231029200959809.png)

![image-20231029201101183](../images/image-20231029201101183.png)



### 集合框架

Java集合框架提供了⼀套性能优良、使⽤⽅法简单的接⼝和类，它们位于 java.util 包中。

collection接口

![image-20231029203855145](../images/image-20231029203855145.png)

Map接⼝：存储⼀组键值对象，提供 key 到 value 的映射

![image-20231029204024239](../images/image-20231029204024239.png)

![image-20231029204052660](../images/image-20231029204052660.png)





# Maven

## 是什么

> Maven是一个Java项目管理和构建工具，它可以定义项目结构、项目依赖，并使用统一的方式进行自动化构建，是Java项目不可缺少的工具。
>

## 作用

### 依赖管理

![image-20230426102535777](E:\typora img\image-20230426102535777.png)

### 模块化管理

把一个项目分成多个模块

![image-20230426102935510](E:\typora img\image-20230426102935510.png)

Maven的中央仓库，存储各种模块，例如，我们使用commons logging、log4j这些第三方模块，就是第三方模块的开发者自己把编译好的jar包发布到Maven的中央仓库中。



## 更换中央仓库源

修改 maven 根目录下的 conf 文件夹中的 settings.xml 文件，在 mirrors 节点上，添加内容如下：

```xml
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```



## 插件 

https://www.runoob.com/maven/maven-plugins.html



## pom.xml

https://www.runoob.com/maven/maven-pom.html

包含了项目的基本信息，用于描述项目如何构建，声明项目依赖，等等。

引入外部依赖

> 要添加依赖项，我们一般是先在 src 文件夹下添加 lib 文件夹，然后将你工程需要的 jar 文件复制到 lib 文件夹下。
>

### 注册依赖

```xml
<dependencies>
    <!-- 在这里添加你的依赖 -->
    <dependency>
        <groupId>ldapjdk</groupId>  <!-- 库名称，也可以自定义 -->
        <artifactId>ldapjdk</artifactId>    <!--库名称，也可以自定义-->
        <version>1.0</version> <!--版本号-->
        <scope>system</scope> <!--作用域-->
        <systemPath>${basedir}\src\lib\ldapjdk.jar</systemPath> <!--项目根目录下的lib文件夹下-->
    </dependency> 
</dependencies>
```



## 项目结构

![image-20230426105518262](E:\typora img\image-20230426105518262.png)



## 与Gradle的区别

https://www.runoob.com/maven/maven-plugins.html



# JAVA Agent

# 内存马

[内存马探究-安全客 - 安全资讯平台](https://www.anquanke.com/post/id/279160#h3-1)

[Java安全-Tomcat内存马学习-信息安全博客](https://trrqsec.github.io/2022/04/10/Java安全-Tomcat内存马学习/)

[Tomcat 内存马学习(一)：Filter型 – 天下大木头](http://wjlshare.com/archives/1529)

[Tomcat Filter 型内存马流程理解与手写 EXP - FreeBuf网络安全行业门户](https://www.freebuf.com/articles/web/343105.html)

[Java Web(一) Servlet详解！！ - 有梦想的老王 - 博客园](https://www.cnblogs.com/whgk/p/6399262.html)



## 前置知识

![在这里插入图片描述](E:\typora img\201909251250360.png)

> 客户端发起一个http请求，比如get类型。
>
> Tomcat 作为Servlet容器,将http请求文本接收并解析，然后封装成HttpServletRequest类型的request对象，传递给servlet；
>
> Servlet容器接收到请求，根据请求信息，封装成HttpServletRequest和HttpServletResponse对象。
> Servlet容器调用HttpServlet的init()方法，init方法只在第一次请求的时候被调用。
> Servlet容器调用service()方法。
> service()方法根据请求类型，这里是get类型，分别调用doGet或者doPost方法，这里调用doGet方法。
> doXXX方法中是我们自己写的业务逻辑。
>
> 业务逻辑处理完成之后，返回给Servlet容器，Servlet容器会将响应的信息封装为HttpServletResponse类型的response对象，然后将response交给tomcat，tomcat就会将其变成响应文本的格式发送给浏览器，然后容器将结果返回给客户端。
>
> 容器关闭时候，会调用destory方法



Tomcat的配置

![img](E:\typora img\v2-c3adc47d4c42dae472308ca19eb84374_720w.webp)

- Server 元素表示整个 Catalina servlet 容器。
- Service元素表示一个或多个连接器组件的组合，这些组件共享一个用于处理传入请求的引擎组件。Server 中可以有多个 Service。
- Executor表示可以在Tomcat中的组件之间共享的线程池。
- Connector代表连接组件。Tomcat 支持三种协议：HTTP/1.1、HTTP/2.0、AJP。
- Context元素表示一个Web应用程序，它在特定的虚拟主机中运行。每个Web应用程序都基于Web应用程序存档（WAR）文件，或者包含相应的解包内容的相应目录，如Servlet规范述。
- Engine元素表示与特定的Catalina服务相关联的整个请求处理机器。它接收并处理来自一个或多个连接器的所有请求，并将完成的响应返回给连接器，以便最终传输回客户端。
- Host元素表示一个虚拟主机，它是一个服务器的网络名称（如“www.mycompany.com”）与运行Tomcat的特定服务器的关联。

![img](E:\typora img\1622447564_60b495cc4ddf6ab5c7b98.png!small)





```java
Engine，实现类为 org.apache.catalina.core.StandardEngine
Host，实现类为 org.apache.catalina.core.StandardHost
Context，实现类为 org.apache.catalina.core.StandardContext
Wrapper，实现类为 org.apache.catalina.core.StandardWrapper
```

Tomcat启动加载的顺序：Listener -> Filter -> Servlet





## 传统web应用型

### listener类型

最终实例化出来的监听器被存储在 `applicationEventListenersList` 属性中

所以内存马构造中需要获取到 StandardContext 类，然后获取其applicationEventListenersList 属性，最后注入我们构造的恶意监听器到 applicationEventListenersList 属性里





### filter类型

[Tomcat 内存马学习(一)：Filter型 – 天下大木头](http://wjlshare.com/archives/1529)

**动态注册恶意 Filter，并且将其放到 `filterChain `的最前面**

#### filter过滤器工作原理

![image-20210331212905616](E:\typora img\image-20210331212905616.png)

1. 根据请求的 URL 从 `FilterMaps` 中遍历`FilterMap`找出与之 URL 对应的 Filter 名称
2. 根据 Filter 名称去 `FilterConfigs` 中寻找对应名称的 `FilterConfig`
3. 找到对应的 `FilterConfig` 之后添加到` FilterChain`中，并且返回 `FilterChain`
4. `filterChain `中调用 `internalDoFilter` 遍历获取 chain 中的 `FilterConfig `，然后从` FilterConfig` 中获取 Filter，然后调用 Filter 的 doFilter 方法

> 关键点在于`FilterMap`和`FilterConfig`，如果我们可以控制这两个变量，我们就可以任意注册filter。
>
> 他们存储在`StandardContext`的属性`FilterMaps`和`FilterConfigs`中，而`StandardContext`是可以获得的，`filterconfig`中需要添加的`filterdef`也是我们可控的，综上filter动态添加过程是我们可操控的。

![image-20230424230657970](E:\typora img\image-20230424230657970.png)

- `filterConfigs`

  ```
  存放filterConfig的数组，在 FilterConfig 中主要存放 FilterDef 和 Filter对象等信息
  ```

- `filterDefs`

  ```
  存放FilterDef的数组 ，FilterDef 中存储着我们过滤器名，过滤器实例，作用 url 等基本信息
  ```

- `filterMaps`

  ```
  存放FilterMap的数组，在 FilterMap 中主要存放了 FilterName 和 对应的URLPattern
  ```

  

思路：

1. 创建一个恶意 Filter
2. 利用 FilterDef 对 Filter 进行一个封装
3. 将 FilterDef 添加到 FilterDefs 和 FilterConfig
4. 创建 FilterMap ，将我们的 Filter 和 urlpattern 相对应，存放到 filterMaps中（由于 Filter 生效会有一个先后顺序，所以我们一般都是放在最前面，让我们的 Filter 最先触发）

<img src="E:\typora img\image-20230424231859397.png" alt="image-20230424231859397" style="zoom: 150%;" />





### servlet类型

工作流程

```
装载：启动服务器时加载Servlet的实例

初始化：web服务器启动时或web服务器接收到请求时，或者两者之间的某个时刻启动。初始化工作有init()方法负责执行完成

调用：即每次调用Servlet的service()，从第一次到以后的多次访问，都是只是调用doGet()或doPost()方法（doGet、doPost内部实现，具体参照HttpServlet类service()的重写）

销毁：停止服务器时调用destroy()方法，销毁实例
```

Servlet需要重写doGet和doPost

```
1. 通过 context.createWapper() 创建 Wapper 对象；

2. 设置 Servlet 的 LoadOnStartUp 的值；（优先级）

3. 设置 Servlet 的 Name；

4. 设置 Servlet 对应的 Class；

5. 将 Servlet 添加到 context 的 children 中；

6. 将 url 路径和 servlet 类做映射。
```







## 排查

阿里巴巴的Java诊断工具--阿尔萨斯

注意启动用的java版本要和启动tomcat的java版本一致，用户一致。

```shell
# servelt
mbean | grep "name=/"


# filter
sc *.Filter # 模糊搜索，列出所有调用了 Filter 的类
sc -d org.apache.coyote.module.SimpleSerializers # classloader

classloader # 查看类加载器加载了哪些类

jad --source-only org.apache.jsp.evil_jsp # 反编译类


# 终极排查 heapdump
heapdump
strings /var/cache/tomcat/temp/heapdump2022-10-19-12-464292342944555007800.hprof | grep "POST /"
```



![image-20230503204528006](E:\typora img\image-20230503204528006.png)

![image-20230426163320217](E:\typora img\image-20230426163320217.png)

动态注册，对比`web.xml`中的配置







# 报错解决

[(69条消息) Intellij idea 报错：Error : java 不支持发行版本5_idea错误不支持发行版本5_灵颖桥人的博客-CSDN博客](https://blog.csdn.net/qq_22076345/article/details/82392236)