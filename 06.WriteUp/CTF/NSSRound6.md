---
title: NSSRound 6
date: 2023/5/31 19:48:25
categories:
- CTF
---



# [NSSRound#6 Team]check

## V1--任意文件上传

> 服务端对上传的压缩包进行解压操作，并且没有对压缩包内容进行检查，导致可以通过符号链接达到任意覆盖/上传文件的效果。

### 例子

假设我们要覆盖/home/khaz/shell目录下的1.sh文件

![image-20221020234420015](../images/image-20221020234420015-1686150537835.png)

那么我们首先可以先创建一个软链接指向这个目录，并将其打包成压缩包1。

![image-20221020234340481](../images/image-20221020234340481-1686150537835.png)

然后我们需要再创建一个与软链接名字相同的目录，并在这个目录下创建要覆盖的文件。然后将该目录打包为压缩包2。

![image-20221020234743494](../images/image-20221020234743494-1686150537835.png)

然后解压压缩包1，得到test，然后解压压缩包2，其中test/clean.sh，因为当前目录下test是指向/home/khaz/shell的，所以实际上会将clean.sh解压到/home/khaz/shell下从而覆盖原来的clean.sh。

![image-20221020234513190](../images/image-20221020234513190-1686150537835.png)



```python
try:
        tar = tarfile.open(file_save_path, "r")
        tar.extractall(app.config['UPLOAD_FOLDER'])
    
@app.route('/clean', methods=['POST'])
def clean_file():
    os.system('/tmp/clean.sh')
    return 'success'

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True, port=80)
```

做的时候，发现反弹shell不生效，在本地测试后发现是因为没有给clean.sh权限，所以覆盖后就不能执行了。

![Snipaste_2022-10-21_18-51-12](../images/Snipaste_2022-10-21_18-51-12-1686150537835.png)



ps：这里还有一个思路就是既然可以任意上传/覆盖文件，那么可以直接覆盖app.py文件。

​		这是因为每次修改工作目录下的文件，flask都会自动重载文件。

![image-20221021202646267](../images/image-20221021202646267-1686150537835.png)



## V2--任意文件下载

> 服务端对上传的压缩包进行解压操作，并且没有对压缩包内容进行检查，并提供下载功能。导致解压出来的文件如果是符号链接文件就可以下载任意文件。

```python
@app.route('/download', methods=['POST'])
def download_file():
    filename = request.form.get('filename')
    if filename is None or filename == '':
        return '?'
    
    filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)
    
    if '..' in filename or '/' in filename:
        return '?'
    
    if not os.path.exists(filepath) or not os.path.isfile(filepath):
        return '?'
    
    with open(filepath, 'r') as f:
        return f.read()

@app.route('/clean', methods=['POST'])
def clean_file():
    os.system('su ctf -c /tmp/clean.sh')
    return 'success'

if __name__ == '__main__':
		app.run(host='0.0.0.0', debug=True, port=80)
```

与v1相比不同点在于`su ctf -c /tmp/clean.sh`限制了用户权限，但是flask还是以root权限启动的，所以下载功能还是root权限。

构造符号链接指向`/flag`，利用下载功能就能够读取到flag

```python
import requests 

url = "http://43.142.108.3:28187/download"

params = {
    "filename": "test2"
}

res = requests.post(url=url,data=params).text

print(res)
```

