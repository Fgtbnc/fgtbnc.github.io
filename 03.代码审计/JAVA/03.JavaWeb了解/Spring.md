# SSM（SpringMVC + Spring + MyBatis）

- SpringMVC代替Struts2

- MyBatis代替Hibernate

- Spring并不是一个新的web框架，而是作者认为java的框架已经足够多且功能完善，只需要一个“工具”将他们整合起来即可，因此spring诞生了。

  Spring最重要的两个思想IOC(DI)和AOP：

  * IOC(DI)即控制反转（依赖注入），这里的控制指的是对类实例化的控制，反转是指通常类的实例化是由开发人员完成的，现在将这个权利给予容器（这里就是spring），告诉容器实例化类的方法，在使用时可以直接调用，降低的代码的耦合度和量，同时也增加了程序的可读性。
  * AOP即面向切面编程，简单理解就是将各个模块公用的模块（如打日志）抽出来，也可以降低代码的耦合度。



# Spring Boot

Spring Boot 是对 Spring 工作流层面的演进，实现基于 Spring 的更便捷的生产级别应用。
Spring Boot 的优点：

- 内嵌 Servlet 容器，独立运行的 Spring 项目，Spring Boot 可以内嵌 Tomcat，以 java -jar xx.jar 包的形式来运行一个 Spring Boot 应用，省略了 war 包部署的繁琐。
- 提供 starter 机制简化 Maven 依赖管理与 Spring 配置，如仅需要引入“spring-boot-starter-web”就可以包含 Spring 和 SpringMVC 相关的依赖和配置。

## 项目结构

![image-20231026152523289](D:/images/image-20231026152523289.png)

[过滤器（Filter）和 拦截器（Interceptor ) 的区别_filter_security_interceptor-CSDN博客](https://blog.csdn.net/SThranduil/article/details/86680531)

另外还要注意`POM.xml`

- `pom.xml`： Maven 构建说明文件（管理项目依赖）

  spring-boot项目特有的标签

  ```xml
  <parent>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-parent</artifactId>
  		<version>2.2.2.RELEASE</version>
  		<relativePath/> <!-- lookup parent from repository -->
  </parent>
  ```

  > 有了这个，当前的项目才是 Spring Boot 项目，spring-boot-starter-parent 是一个特殊的 starter ，它用来提供相关的Maven默认依赖。**使用它之后，常用的包依赖就可以省去 version 标签，关于具体 Spring Boot 提供了哪些 jar 包的依赖，我们可以查看本地 Maven 仓库下：\repository\org\springframework\boot\spring-boot-dependencies\2.2.2.RELEASE\spring-boot-dependencies-2.2.2.RELEASE.pom 文件来查看**

  比如说spring boot内置了Tomcat作为默认的Servlet容器，2.2.2版本对应的Tomcat版本为9.0.29。

  > ![image-20230930202303209](D:/images/image-20230930202303209.png)

- **项目配置文件**在`src/main/resources`目录下

  `application.properties`或者`application.yml`

  ![img](D:/images/2019123117071828.png)

- **应用入口类**

  > Spring Boot 项目通常有一个名为 *Application 的入口类，入口类里有一个 main 方法， **这个 main 方法其实就是一个标准的 Java 应用的入口方法。**

## 请求传递流程

![request-path](D:/images/request-path.png)

为了更好理解，以「保存订单」功能为例，主要的请求流程如下图，不了解Spring请求传递的同学可以在代码中跟一遍请求流程，会加深请求传递的印象。

![request-example](D:/images/request-example.png)



## 注解

```bash
@SpringBootApplication: 用于标识主类，表明这是一个 Spring Boot 应用程序的入口点。它组合了 @Configuration、@EnableAutoConfiguration 和 @ComponentScan 注解。

@RestController: 标识一个类为 RESTful 风格的控制器，用于处理 HTTP 请求并返回响应。

@RequestMapping: 定义请求的 URL 路径，可用于类级别或方法级别。用于映射 HTTP 请求到相应的控制器方法。

@GetMapping, @PostMapping, @PutMapping, @DeleteMapping: 分别用于标识 GET、POST、PUT 和 DELETE 请求的处理方法。

@RequestBody: 用于将请求体的内容绑定到方法参数上，通常用于接收 JSON 或 XML 数据。

@PathVariable: 用于从 URL 路径中获取变量值，并将其绑定到方法参数上。

@RequestParam: 用于获取请求参数的值，并将其绑定到方法参数上。

@Autowired: 自动装配依赖项，通过类型来查找匹配的 bean，并注入到字段、构造函数或方法参数中。

@Value: 用于将属性值注入到字段、构造函数或方法参数中。

@Configuration: 声明一个类作为配置类，用于定义 bean 的创建和配置。

@Component: 标识一个类为 Spring 组件，会被自动扫描并创建为 bean。

@Service: 标识一个类为服务层组件，通常用于注解业务逻辑的实现类。

@Repository: 标识一个类为数据访问层组件，通常用于注解数据库操作的实现类。

@EnableAutoConfiguration: 开启自动配置功能，Spring Boot 会根据项目的依赖和配置自动进行相应的配置。

@Conditional: 根据条件来决定是否创建某个 bean 或执行某段代码。
```

在`application.yml`中添加

```yaml
server:
  port: 8080
  servlet:
    context-path: /

name: Khaz
age: 18

content: "name${name},age${age}"
```

新建`HelloController`类

```java
@RestController
public class HelloController {

    @Value("${age}")
    private Integer age;

    @Value("${name}")
    private String name;

    @RequestMapping(value = "/hello")
    public String get(){
        return name + ":"+age;
    }

}
```

`@RestController`所以数据格式为json格式

![image-20230925113401628](D:/images/image-20230925113401628.png)

## 运行jsp

**依赖**

```xml
		<!--jsp-->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <version>8.5.28</version>
        </dependency>
        <!--servlet-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
        </dependency>
        <!--jstl-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
        </dependency>
```

**主流的目录结构**

> 非必须，配置好视图路径前缀即可

![image-20230925115020412](D:/images/image-20230925115020412.png)

```
└─WEBAPP
    └─WEB-INF
        └─jsp
            └─index.jsp
```

![image-20230925115955133](D:/images/image-20230925115955133.png)

**配置视图**

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp
```

> prefix需要加上/，路径的拼接为前缀+视图名称+后缀：/WEB-INF/jsp/index.jsp

**控制类**

```java
@Controller
@RequestMapping("web")
public class WebController {
    @RequestMapping(value = "index",method = RequestMethod.GET)
    public String index(){
        return "index";
    }
}
```

- 通过`@Controller`注解将`WebController`类声明为Spring MVC的控制器。
- 使用`@RequestMapping("web")`注解将`web`作为基本URL路径，表示该控制器处理以`/web`开头的请求。
- 在`index`方法上使用`@RequestMapping(value = "index", method = RequestMethod.GET)`注解来映射具体的请求路径，即处理以`/web/index`的GET请求。
- 方法返回一个字符串 `"index"`，表示要渲染名为`index`的视图。

**运行结果**

![image-20230925120058229](D:/images/image-20230925120058229.png)





# SpringCloud (了解即可)







# 参考文章

[通过IDEA搭建springboot项目（maven方式）_idea创建springboot +maven_智Min的博客-CSDN博客](https://blog.csdn.net/hezhimin1124/article/details/103782745)

https://congzhou09.github.io/knowledge/Java-web-%E6%8A%80%E6%9C%AF%E4%B8%8E%E6%9E%B6%E6%9E%84%E6%BC%94%E8%BF%9B%E5%8E%86%E5%8F%B2.html