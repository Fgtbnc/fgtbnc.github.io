# Servlet Tomcat JSP

- Tomcat: 是Servlet的容器,Servlet运行在Tomcat容器当中,Servlet容器有很多(tomcat,Jboss,jetty)等

- JSP: 全称Java Server Pages，是一种动态网页开发技术。它使用JSP标签在HTML网页中插入Java代码。标签通常以<%开头以%>结束。

- Serlvet: 全称Java Servlet,是用Java编写的服务器端程序。Servlet只是一个Java类，就像控制器类一样，接收前端传过来的数据，然后进行处理。

  > 定义一个 Servlet 很简单，只需要继承`javax.servlet.http.HttpServlet`类并重写`doXXX`(如`doGet、doPost`)方法或者`service`方法就可以了
  >
  > ![image-20231020191711886](D:/images/image-20231020191711886.png)

  

## 基于Web.xml配置--Servlet<3.0

![在这里插入图片描述](../../../../../../../知识库/blog/source/_posts/代码审计/images/201909251250360.png)

页面和类是毫无关联的，需要在web.xml里面创建Url与Servlet之间的映射关系，`/login`由`FirstServlet`来进行处理

![image-20230925101303360](D:/images/image-20230925101303360.png)

**简单实现用户登录判断**

FirstServelt

```java
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class FirstServlet extends HttpServlet {
    
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String name = request.getParameter("name");
        String password = request.getParameter("password");

        String html = null;

        if ("admin".equals(name) && "123".equals(password))
            html = "<div style='color:green'>success</div>";
        else
            html = "<div style='color:red'>fail</div>";

        PrintWriter pw = response.getWriter();
        pw.println(html);
    }
}
```

index.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
  <title>$Title$</title>
</head>
<body>

<form action="login" method="post">
  账号: <input type="text" name="name"> <br>
  密码: <input type="password" name="password"> <br>
  <input type="submit" value="登录">
</form>

</body>
</html>
```

![image-20230925101208723](D:/images/image-20230925101208723.png)

![image-20230925101409954](D:/images/image-20230925101409954.png)

请求的处理流程：前端访问action，然后在web.xml会根据同名的url-pattern去访问对应的servlet类，servlet类里面做完相应的处理后，再返回内容到页面上



## 基于注解--Servlet>=3.0

```java
@WebServlet(name = "FirstServlet", urlPatterns = {"/login"})
```

```java
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet(name = "FirstServlet", urlPatterns = {"/login"})
public class FirstServlet extends HttpServlet {
    
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String name = request.getParameter("name");
        String password = request.getParameter("password");

        String html = null;

        if ("admin".equals(name) && "123".equals(password))
            html = "<div style='color:green'>success</div>";
        else
            html = "<div style='color:red'>fail</div>";

        PrintWriter pw = response.getWriter();
        pw.println(html);
    }
}
```

