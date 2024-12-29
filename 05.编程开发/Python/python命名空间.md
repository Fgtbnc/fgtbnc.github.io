---
title: python命名空间
date: 2023/5/1 15:00:25
categories:
- Python
---

From：https://www.bilibili.com/video/BV12F411t78n

开头说了一段话

```text
lf you import another module, it will have its global scope
每个模块都有自己的全局空间
Each function has its local scope
每个函数都有自己的局部空间
Every time the function is called, a new scope is created
每当函数被调用时，就会创建一个空间给它使用
Certain Python objects like 'print' function, 'None','True', 'False', are available everywhere in your program and they are in the built-in scope
python内置的函数在程序的任何一个空间都是可以使用的，它们存在于built-in空间中。
The variables or name bindings are stored in namespaces
变量和函数地址都存储在命名空间中
```

这是我根据视频的画的，左边的global空间是socket模块的，右边的global空间是我们程序的（我给它起了叫original）

<img src="E:\typora img\Untitled-2022-08-08-1428.png" alt="Untitled-2022-08-08-1428" style="zoom:80%;" />

所以我们的程序处在orginal里，所以想用socket模块中的方法就必须用.来访问

```python
import socket
#创建TCP/IP套接字
tcpSock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
#创建UDP/IP套接字
udpSock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
```

但是我们像下面这样写，就相当于把socket模块导入到我们的命名空间original中，就可以直接调用方法了

```python
from socket import *
#创建TCP/IP套接字
tcpSock = socket(AF_INET, SOCK_STREAM)
#创建UDP/IP套接字
udpSock = socket(AF_INET, SOCK_DGRAM)
```









什么都没有的py

```python
def test():
    a=10
    print(locals())#查看test函数所在的local空间
    
print(globals())#查看global空间
print("\n")	
test()
print("\n")
print(dir(__builtins__))#查看built-in空间（就是python的内置函数和模块）
```

![image-20220808144812676](../images/image-20220808144812676-1687707604251.png)

```python
#global
{'__name__': '__main__',
 '__doc__': None,
 '__package__': None,
 '__loader__': <_frozen_importlib_external.SourceFileLoader object at 0x0000023A5A4A0850>,  '__spec__': None, 
 '__annotations__': {}, 
 '__builtins__': <module 'builtins' (built-in)>,  
 '__file__': 'd:\\python3.8\\code\\1.py', 
 '__cached__': None, 
 'test': <function test at 0x0000023A5A5E9F70>}
```





![image-20220808140251398](../images/image-20220808140251398-1687707604252.png)

因为在my_func中先使用了变量a`print(f"iprefix} {a}")`,然后才在my_func中声明了变量a,所以报错。





![image-20220808140402575](../images/image-20220808140402575-1687707604252.png)

在my_func中声明了`global a`,所以python知道local空间中的a指向global空间中的a。

所以在local中改变a的值，会影响global中的a。





![image-20220808140452593](../images/image-20220808140452593-1687707604252.png)

原本在global空间中没有变量a,但是在local中声明了`global a`，python会自动在global中分配空间给变量a。

> 作者说it's bad code😄



![image-20220808140611573](../images/image-20220808140611573-1687707604252.png)

`globals()`返回的是字典形式，可以通过键值索引的方式访问。



![image-20220808140711338](../images/image-20220808140711338-1687707604252.png)

如何展示函数的local空间中保存的变量

1. 函数的body中`print(locals())`

2. 通过函数的code方法下的属性访问

   ![image-20220808151843058](../images/image-20220808151843058-1687707604252.png)









![image-20220808153058120](../images/image-20220808153058120-1687707604253.png)

```python
a=10
def test():
	x=20
	def test2():
		global x
		x=10
	test2()
	print(x)

test()
print(x)
```

![image-20220808141136207](../images/image-20220808141136207-1687707604253.png)





![image-20220808141228174](../images/image-20220808141228174-1687707604253.png)

nonlocal用于寻找同为local空间的变量



```
nonlocal means, not in the current function's local scope and neither in the global scope
nonlocal的寻找范围：上一级local空间到global空间的下一级
```

![image-20220808141525359](../images/image-20220808141525359-1687707604253.png)

![image-20220808141538079](../images/image-20220808141538079-1687707604253.png)

虽然global中声明了变量a，但是nonlocal还是找不到变量a



![image-20220808141845300](../images/image-20220808141845300-1687707604253.png)

![image-20220808141538079](../images/image-20220808141538079-1687707604253.png)

即使在local中声明了`global a`,nonlocal仍然找不到变量a

所以nonlocal的寻找范围：上一级local空间到global空间的下一级