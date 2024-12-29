---
title: MISC
date: 2023/5/1 19:48:25
categories:
- CTF
---

wiki：https://cuccs.github.io/ctf-wiki/MISC/MISC-2/MISC-2/

编码：https://cloud.tencent.com/developer/article/1749430

奇奇怪怪解密：https://www.dcode.fr/

brain_fuck:https://www.splitbrain.org/services/ook

misc高手博客：http://www.ga1axy.top/index.php/archives/6

 https://zhouhaobusy.com/

jphide：[【隐写工具】【试一试？】jphide seek（JPHS） 使用方法,检测提示，附下载地址_黑色地带(崛起)的博客-CSDN博客_jphs使用](https://blog.csdn.net/qq_53079406/article/details/123608345)

usb流量分析：[USB流量分析](https://atkx.top/2021/04/12/USB%E6%B5%81%E9%87%8F%E5%88%86%E6%9E%90/#%E9%94%AE%E7%9B%98%E6%B5%81%E9%87%8F)

拼图教程：https://blog.csdn.net/qq_51999772/article/details/122425898

还原拼图gaps安装：https://blog.csdn.net/ZT7524/article/details/119981204

gaps的size参数=属性中的像素值/碎片个数

二维码https://mp.weixin.qq.com/s/1C98fhfoP81onob6a_v6Ag

二维码扫描工具：QRCODE，手机app  CortexScan

二维码/条形码在线：https://online-barcode-reader.inliteresearch.com/

摩斯电码：http://www.zhongguosou.com/zonghe/moersicodeconverter.aspx



> flag base64 编码后 ZmxhZw
>







图片详细信息

直接在010内搜索flag



#### linux命令

- file
- binwalk
- foremost
- strings
- grep -a strings  filename
- exiftool







#### 常见文件格式

https://www.cnblogs.com/lwy-kitty/p/3928317.html

| 文件格式 |           文件头           |   文件尾    |
| :------: | :------------------------: | :---------: |
|   JPG    |           FF D8            |    FF D9    |
|   PNG    |        89 50 4E 47         | AE 42 60 82 |
|   GIF    |   47 49 46 38（Gif89a）    |    00 3B    |
|   ZIP    |     50 4B 03 04（PK）      |    50 4B    |
|   RAR    |    52 61 72 21（Rar!）     |             |
|    7z    |     37 7A BC AF 27 1C      |             |
|   PDF    | %PDF- 加上版本号如%PDF-1.4 |             |

检查后缀名、文件头是否正确

破环文件头，一定不能显示，破环文件尾，文件不一定显示不正常

比如说我可以在图片的文件尾加一个压缩包，图片仍能够正常显示。



#### png

- IHDR

  ```
  脚本爆破，010修改
  ```

- LSB/MSB通道隐写

  ```
  使用zsteg（kali）来分析
  
  #分析每个通道
zsteg file_name
  
  #提取通道
  zeteg -e '通道' 源文件 > 输出文件
  ```
  
- 反相，red plane等用stegsolve

  <img src="E:\typora img\image-20220922170514788.png" alt="image-20220922170514788" style="zoom:50%;" />



#### jpg

- Stegdetect：探秘加密方式

  ![img](../images/23851034-5903620b52a47f98-1699691014649.png)

- jphide

  ![img](E:\typora img\23851034-02faa6817e259909.png)
  
- outguess

  ```shell
  解密
  outguess -k 密码 -r 文件名 输出文件
  
  加密
  outguess -k 密码 -d 要隐藏的文件 源图片 输出图片
  ```

- F5隐写

  ```shell
  java Extract 图片的绝对路径 -p 密码 -e 输出文件
  ```

  

#### 通用图片隐写

silenteye

可以将文字或者文件隐藏到图片的解密工具









#### gif

- 帧--wps图片







#### zip

##### 伪加密

修改文件内容如下：

50 4B 03 04：头文件标记

其后的第二个字节为00 00



50 4B 01 02：目录中文件文件头标记 （压缩包里有几个文件就有几个这个标志）

其后的第三个字节为09 00（只要09部分为奇数即可）



##### 备注

![image-20220901164337837](../images/image-20220901164337837-1699691018024.png)

##### crc爆破

> CRC是用文件保存的内容作为明文加密的，所以可以根据CRC的不同判断出文件内容是否相同，也可以进行逆向破解得到文件内容。

适用于文件大小不大的

`py crc32.py reverse 文件crc值（十六进制）`



##### 已知明文攻击

有一个单独的文件已知且进行压缩之后的[CRC](https://link.juejin.cn?target=https%3A%2F%2Fso.csdn.net%2Fso%2Fsearch%3Fq%3DCRC%26spm%3D1001.2101.3001.7020)值与某个包含此文件的压缩包的CRC值相等。







# 520迎新赛

## Pngenius

binwalk命令得到zip压缩包，不是伪加密，也不是弱密码，用stegsolve打开在rgb通道末位发现

![image-20220719160854697](../images/image-20220719160854697-1699690319383.png)

或者用zsteg更方便

![image-20220719175149412](../images/image-20220719175149412-1699690319385.png)

解压得到flag



## EasyEncode

压缩包弱密钥爆破，得到16进制,转换得到unicode编码，再转换得到base64编码，解码得到flag

```python
import base64
def hex_to_bin(a):
	b=''
	for i in range(0,len(a),2):
		b+=chr(int(a[i:i+2],16))
	return b


a='5c75303035325c75303034375c75303035365c75303037615c75303036345c75303034345c75303034325c75303036655c75303034645c75303033335c75303037345c75303034355c75303035615c75303035375c75303033395c75303036625c75303036315c75303035375c75303033355c75303036655c75303035385c75303037615c75303034365c75303037615c75303035385c75303033325c75303035355c75303033305c75303036335c75303033335c75303036635c75303036365c75303034655c75303034365c75303033395c75303035365c75303036365c75303035315c75303032355c75303033335c75303034345c75303032355c75303033335c7530303434'
b=hex_to_bin(a)
b=b.replace('\\u00','')
b=hex_to_bin(b).replace('%3D','=')
print(b)
print(base64.b64decode(b))
```



## 你知道js吗

看到pk开头以为是zip，解压后得到XML，用file命令查看类型得知是word文档

![image-20220719170336929](../images/image-20220719170336929-1699690319384.png)

修改后缀名为.doc，打开

![image-20220719170245328](../images/image-20220719170245328-1699690319384.png)

更改字体或者复制出去会发现其实是base664编码



## StrangeTraffic

不理解

wp写的是

追踪tcp流，寄存器在变化？观察发现base64编码后的Dest。。



不过提供了一个思路就是可以观察字母是否是flag的base64编码





## 4096

https://blog.csdn.net/weixin_46081055/article/details/125046554

到了音频那就不会了

注意的点：

gaps拼图的命令

```shell
gaps --image=part_flag.jpg --size=64 --save 
```

size

> 图片大小：960x1024 像素
>
> 得到的碎片图：15x16个碎片
>
> size=960/15



## Python_jail

空白字符隐写

lsb通道隐写

得到py文件

运行py文件得到flag







# 2021安洵杯

## love math

打开附件看到flag压缩包和几个txt文件

![image-20221006161048265](../images/image-20221006161048265-1699690344437.png)

crc爆破得到解压密码

解压得到一张图片，lsb隐写，提取后得到一张图片，只是破坏了文件头，修复之后

根据提示画出自己找到解密网站[Tupper's Self-Referential Formula Playground](http://keelyhill.github.io/tuppers-formula/)

![Tupper-plot](../images/Tupper-plot-1699690344438.png)



## CyzCC_loves_LOL

lol解密得到解压密码

解压后得到两张图片

一张brainfuck加密

另一张silent eye加密

解密brainfuck图片[Brainloller](https://minond.xyz/brainloller/)得到密码

解密silent eye得到flag