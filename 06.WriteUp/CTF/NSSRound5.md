---
title: NSSRound 5
date: 2023/5/31 19:48:25
categories:
- CTF
---

# [NSSRound#V Team]PYRCE

```python
from flask import Flask, request, make_response
import uuid
import os

# flag in /flag
app = Flask(__name__)

def waf(rce):
    black_list = '01233456789un/|{}*!;@#\n`~\'\"><=+-_ '
    for black in black_list:
        if black in rce:
            return False
    return True

@app.route('/', methods=['GET'])
def index():
    if request.args.get("Ňśś"):
        nss = request.args.get("Ňśś")
        if waf(nss):
            os.popen(nss)
        else:
            return "waf"
    return "/source"


@app.route('/source', methods=['GET'])
def source():
    src = open("app.py", 'rb').read()
    return src

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=False, port=8080)
```

思路一，用/flag覆盖app.py文件，再访问/source路由

这里覆盖完了之后，还可以访问路由的原因是`debug=False`，所以即使覆盖了app.py文件，但是flask运行所使用的文件并没有受到影响。

而如果`debug=True`，当我们修改文件时，flask就会自动重载文件。

此时覆盖flask运行文件（我本地是pyrce.py）

控制台

![image-20221022132313611](../images/image-20221022132313611-1686150585118.png)

访问/source路由

![image-20221022132349822](../images/image-20221022132349822-1686150585118.png)



思路二，利用可以访问静态文件，先创建static目录（flask框架结构，静态文件放在static目录下），再将flag复制到static目录，访问/static/flag就能够下载flag文件。











