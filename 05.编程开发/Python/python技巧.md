---
title: python技巧
date: 2023/5/1 15:00:25
categories:
- Python
---

### 输出列表元素

```python
lis=[1,2,3]
print(lis[1:])
print(*lis[1:])
>>
[2, 3]
2 3
```



### 同时获得元素与元素索引

```python
lis = ['a','b','c']

for index,i in enumerate(lis):
	print(index,i)
>>
0 a
1 b
2 c
```

### 字符串格式化

- 类似c语言

  ```python
  # %s 字符串占位
  print("%s"%'abcd')
  # %c 字符占位
  print("%c"%'a')
  ```

- format   (会自动将变量转为str)

  ```python
  print("{}{}".format(1,2))
  a=1
  b=2
  print(f'{a}{b}')
  >>
  12
  12
  ```

  > 占位符格式化https://blog.csdn.net/u014770372/article/details/76021988



### 交换数值

```python
a=1
b=2
a,b=b,a
print(a,b)
>>2 1
```



### 推导式

> [想得到的变量形式 for 变量 in 可迭代对象 if 条件表达式]

```python
#找出列表里大于0的数
shu=[1,2,3,-1,-2,-3]
print([i for i in shu if i>0])
>>[1,2,3]

#统计一个字符串中每个字符出现的次数
m = 'I am Khazking!'
n = {i:m.count(i) for i in m}
print(n)
>>{'I': 1, ' ': 2, 'a': 2, 'm': 1, 'K': 1, 'h': 1, 'z': 1, 'k': 1, 'i': 1, 'n': 1, 'g': 1, '!': 1}
```



### 字符串替换

```python
#repalce方法只能实现元素一对一替换
#使用下述方法可以实现b替换为z,z替换为b
#建立映射关系字典
a='zbbz'
table = ''.maketrans('bz','zb')#等价于 table= {'b':'z','z':'b'}
a = a.translate(table)
print(a)
>>bzzb
```



### 各种排序

`reverse=True`降序，默认为升序

- 字典排序

  ```python
  # 按键值排序
  sorted(dic.items(),key=lambda item:item[1],reverse=True)
  # 按键名排序
  sorted(dic.items(),key=lambda item:item[0],reverse=True)
  ```

- 嵌套排序

  ```python
  # 字典嵌套
  {'a': [1, 3], 'c': [3, 4], 'b': [0, 2], 'd': [2, 1]}
  sorted(dic.items(), key=lambda x: x[1][1], reverse=True) # 按键值（列表）的第二个元素排列
  # 列表嵌套
  [['金牛', 89000, 13140.196950444726, 2800, 1574], ['锦江', 113800, 18719.342387419587, 2550, 1399], ['成华', 96400, 14034.861538461539, 2300, 1300], ['高新', 160000, 
  23663.31254871395, 3600, 1283], ['武侯', 102000, 19768.739789964995, 3100, 857]]
  sorted(lis, key = lambda k : k[2],reverse= True) # 按每个子列表的第三个元素排列
  ```

修改下标即可按照其他元素进行排列。





### 统计字符出现的频率

```python
from random import randint
from collections import Counter

data = [randint(0,20) for i in range(30)]

s = dict.fromkeys(data,0)
print(s)

for i in data:
    s[i]+=1
    
# 统计字符出现次数
print(s)
    
#统计s中出现频率最高的三个数
print (Counter(s).most_common(3))
```



### 从序列中筛选出符合条件的元素

```python
from random import randint

#filter(function, iterable)
data = [randint(-10,10) for i in range(10)]
data = list(filter(lambda x:x>=0,data))
print(data)

#或者列表推导式
```



### 临时文件

> 我们采集数据进行分析，但是我们只需要保存结果，我们采集的数据如果常驻内存就会让电脑崩溃，于是我们将这个数据放在临时文件中（外部存储），在文件关闭后将被删除

```python
from tempfile import TemporaryFile,NamedTemporaryFile
f = TemporaryFile()
f.write(b'abc'*10000)
f.seek(0)
a=f.read(100)
print(a)
```

> 这个临时文件在系统中是找不到的，如果我们想创建一个能在文件系统中看到的临时文件，我们就用NamedTemporaryFile

```python
nte = NamedTemporaryFile(delete=false)#delete参数为false时程序运行完，临时文件也不会被删除
print(nte.name)
```



### 穷举组合

```python
#全排列  
from itertools import permutations
#permutations()返回的是tuple类型，所要要用''.join(i)连接起来
for i in permutations('123',2):
    print(''.join(i),end=' ')
```





### 模拟终端

```python
import pty;pty.spawn('/bin/bash')
```



### 二进制文件原地修改

```python
import os
import mmap

def memory_map(filename, access=mmap.ACCESS_WRITE):
    size = os.path.getsize(filename)
    fd = os.open(filename, os.O_RDWR)
    return mmap.mmap(fd, size, access=access)
m = memory_map('data')


>>> len(m)
1000000
>>> m[0:10]
b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
```



### 脚本参数

```python
python xx.py aa

sys.argv[0]→xx.py
sys.argv[1]→aa
```

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("-u","--url", type=str, help="target url")
parser.add_argument("-t", "--threads" , type=int, help="threads num")
args = parser.parse_args()

>>
python xx.py -u xx.com -t 20

args.url=xx.com
args.threads=20
```

