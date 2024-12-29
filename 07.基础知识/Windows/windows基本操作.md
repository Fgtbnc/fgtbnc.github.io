# Windows注册表  

## 1、定义  

注册表：

```php
     Windows操作系统称之为登录档案。是Microsoft Windows中的一个重要的数据库，注册表是 windows操作系统中的一个核心数据库，其中存放着各种参数，直接控制着Windows的启动、硬件驱动程序的 装载以及一些Windows应用程序的运行，从而在整个系统中起着核心作用。这些作用包括了软、硬件的相关配置和状态信息，比如注册表中保存有应用程序和资源管理器外壳的初始条件、首选项和卸载数据等，联网计算机的整个系统的设置和各种许可，文件扩展名与应用程序的关联，硬件部件的描述、状态和属性，性能记录和 其他底层的系统状态信息，以及其他数据等。注册表中还包含 Windows 在运行期间不断引用的信息，例如，每个用户的配置文件、计算机上安装的应用程序以及每个应用程序可以创建的文档类型、文件夹和应用程序图 标的属性表设置、系统上存在哪些硬件以及正在使用哪些端口。当一个用户准备运行一个应用程序，注册表提 供应用程序信息给操作系统，这样应用程序可以被找到，正确数据文件的位置被规定，其他设置也都可以被使用。 
     正常情况下，你可以点击开始菜单当中的运行，然后输入regedit或regedit.exe点击确定就能打开 Windows操作系统自带的注册表编辑器了，友情慎重提醒，操作注册表有可能造成系统故障，若您是对 Windows注册表不熟悉、不了解或没有经验的Windows操作系统用户建议尽量不要随意操作注册表，即便是必须要操作，那么也要提前做好注册表的备份工作。如果上述打开注册表的方法不能使用，说明你没有管理员权 限，或者注册表被锁定，如果是没有权限，那么想办法解锁权限。 
     简单来说：注册表是windows系统来记录和修改用户设置的，不论是软件还是硬件。
```

注册表的数据结构：

```
	注册表由项（也叫主键或称“键”）、子项（子键）和值构成。一个项就是分支中的一个文件夹，而子项就
是这个文件夹当中的子文件夹，子项同样它也是一个项。一个值则是一个项的当前定义，由名称、数据类型以
及分配的值组成。一个项可以有一个或多个值，每个值的名称各不相同，如果一个值的名称为空，则该值为该
项的默认值。
	在注册表编辑器（regedit.exe）中，数据结构显示如下，其中，command键是open项的子项，(默
认)表示该值是默认值，值名称为空，其数据类型为REG_SZ，数据值
为%systemroot%/system32/notepad.exe"%1数据类型。
注册表的数据类型主要有以下四种：显示类型（在编辑器中）数据类型说明
    REG_SZ：字符串：文本字符串
    REG_MULTI_SZ：多字符串值：含有多个文本值的字符串
    REG_BINARY：二进制数：二进制值，以十六进制显示，
    REG_DWORD：双字值；一个32位的二进制值，显示为8位的十六进制值。
```

win+r键，运行，输入regedit，即可看到注册表

注册表中关于开机启动项的配置位置在这里 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run

```
对于win10来讲的常见位置：
	HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
 	HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
其他位置：这些大致了解即可，后面用到哪个就说哪个。
    Load注册键：HKEY_CURRENT_USER＼Software＼Microsoft＼WindowsNT＼CurrentVersion＼
    Windows＼load
     Userinit注册键：HKEY_LOCAL_MACHINE＼SOFTWARE＼Microsoft＼WindowsNT＼
    CurrentVersion＼Winlogon＼Userinit这里也能够使系统启动时自动初始化程序。通常该注册键下面有
    一个userinit.exe，但这个键允许指定用逗号分隔的多个程序，例如“userinit.exe,OSA.exe”(不含引
    号)。
    等等..
```

命令行添加注册表选项  

```cmd
reg add HKEY_CURRENT_USER\Software\Valve\Half-Life\Settings /v dms /t REG_SZ /d  test /f 
注：reg 命令;add 增加; /v 选项; /t 类型 ;/d  值;/f  不用提示就强行覆盖现有注册表项
```

## Window安全组策略  

## 1、定义  

百度百科：组策略（英语：Group Policy）是微软Windows NT家族操作系统的一个特性，它可以控制用户 帐户和计算机帐户的工作环境。组策略提供了操作系统、应用程序和活动目录中用户设置的集中化管理和配 置。组策略的其中一个版本名为本地组策略（缩写“LGPO”或“LocalGPO”），这可以在独立且非域的计算机上 管理组策略对象。 

通俗解释：组策略是一组策略的集合，策略就是制定的规则。组策略是将系统重要的配置功能汇集成各种配置 模块，供用户直接使用，从而达到方便管理计算机的目的。简单点说，组策略就是修改注册表中的配置。当 然，组策略使用自己更完善的管理组织方法，可以对各种对象中的设置进行统一的管理和配置，远比手工修改 注册表方便、灵活，功能也更加强大



## 打开本地组策略的方式

win+r键，运行 gpedit.msc