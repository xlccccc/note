## 通过网鼎杯的一道题学习flask框架和yaml反序列化

包含巨多知识点！！！

### 源码

```python
import os
import re
import yaml
import time
import socket
import subprocess
from hashlib import md5
from flask import Flask, render_template, make_response, send_file, request, redirect, session

app = Flask(__name__)
app.config['SECRET_KEY'] = socket.gethostname()

def response(content, status):
    resp = make_response(content, status)
    return resp


@app.before_request
def is_login():
    if request.path == "/upload":
        if session.get('user') != "Administrator":
            return f"<script>alert('Access Denied');window.location.href='/'</script>"
        else:
            return None


@app.route('/', methods=['GET'])
def main():
    if not session.get('user'):
        session['user'] = 'Guest'
    try:
        return render_template('index.html')
    except:
        return response("Not Found.", 404)
    finally:
        try:
            updir = 'static/uploads/' + md5(request.remote_addr.encode()).hexdigest()
            if not session.get('updir'):
                session['updir'] = updir
            if not os.path.exists(updir):
                os.makedirs(updir)
        except:
            return response('Internal Server Error.', 500)


@app.route('/<path:file>', methods=['GET'])
def download(file):
    if session.get('updir'):
        basedir = session.get('updir')
        try:
            path = os.path.join(basedir, file).replace('../', '')
            if os.path.isfile(path):
                return send_file(path)
            else:
                return response("Not Found.", 404)
        except:
            return response("Failed.", 500)


@app.route('/upload', methods=['GET', 'POST'])
def upload():

    if request.method == 'GET':
        return redirect('/')

    if request.method == 'POST':
        uploadFile = request.files['file']
        filename = request.files['file'].filename

        if re.search(r"\.\.|/", filename, re.M|re.I) != None:
            return "<script>alert('Hacker!');window.location.href='/upload'</script>"
        
        filepath = f"{session.get('updir')}/{md5(filename.encode()).hexdigest()}.rar"
        if os.path.exists(filepath):
            return f"<script>alert('The {filename} file has been uploaded');window.location.href='/display?file={filename}'</script>"
        else:
            uploadFile.save(filepath)
        
        extractdir = f"{session.get('updir')}/{filename.split('.')[0]}"
        if not os.path.exists(extractdir):
            os.makedirs(extractdir)

        pStatus = subprocess.Popen(["/usr/bin/unrar", "x", "-o+", filepath, extractdir])
        t_beginning = time.time()  
        seconds_passed = 0
        timeout=60
        while True:  
            if pStatus.poll() is not None:  
                break  
            seconds_passed = time.time() - t_beginning  
            if timeout and seconds_passed > timeout:  
                pStatus.terminate()  
                raise TimeoutError(cmd, timeout)
            time.sleep(0.1)

        rarDatas = {'filename': filename, 'dirs': [], 'files': []}
        
        for dirpath, dirnames, filenames in os.walk(extractdir):
            relative_dirpath = dirpath.split(extractdir)[-1]
            rarDatas['dirs'].append(relative_dirpath)
            for file in filenames:
                rarDatas['files'].append(os.path.join(relative_dirpath, file).split('./')[-1])

        with open(f'fileinfo/{md5(filename.encode()).hexdigest()}.yaml', 'w') as f:
            f.write(yaml.dump(rarDatas))

        return redirect(f'/display?file={filename}')


@app.route('/display', methods=['GET'])
def display():

    filename = request.args.get('file')
    if not filename:
        return response("Not Found.", 404)

    if os.path.exists(f'fileinfo/{md5(filename.encode()).hexdigest()}.yaml'):
        with open(f'fileinfo/{md5(filename.encode()).hexdigest()}.yaml', 'r') as f:
            yamlDatas = f.read()
            if not re.search(r"apply|process|out|system|exec|tuple|flag|\(|\)|\{|\}", yamlDatas, re.M|re.I):
                rarDatas = yaml.load(yamlDatas.strip().strip(b'\x00'.decode()))
                if rarDatas:
                    return render_template('result.html', filename=filename, path=filename.split('.')[0], files=rarDatas['files'])
                else:
                    return response('Internal Server Error.', 500)
            else:
                return response('Forbidden.', 403)
    else:
        return response("Not Found.", 404)


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8888)
```

### flask session伪造

打开环境，根据源码中的upload访问，显示拒绝，看到cookie中的session，很明显第一步就是伪造出一个**Administrator**的session
在前面有一句

![image-20220826191743314](D:\Typora\note\CTF\web\知识点\yaml反序列化.assets\image-20220826191743314.png)

这就是flask中加密session签名部分的密钥，接下来就是想办法得到密钥(hostname 即主机名)

因为本题是文件上传，那么就可能有文件读取，审计源码

![image-20220826191919786](D:\Typora\note\CTF\web\知识点\yaml反序列化.assets\image-20220826191919786.png)

路由是文件路径，试试任意文件读取

**复现一半主办方环境关了！**（才把目录穿越弄出来qaq

题目将**../**替换为空，那么双写就可绕过，然后试了好久，发现两个tips
如果前面的目录真实存在，如**static/../../../../../etc/hosts**
<font color='pink'>**os.path.isfile(path)返回为true**</font>
反之为false，本题明显是有一个不存在的文件夹的，因为你还不能成功上传文件，那怎么办呢？
<font color='red'>**在etc前多加一个/可实现绕过！！**</font>

**本地勉强可试**

### yaml反序列化

经过重重坎坷，终于成了

yaml的漏洞成因不必过度追究，网上的poc已经泛滥了(引用的方法已经危险函数都很清楚)，也有很多绕过的方式，当然最新的版本也已经基本上封死了所以poc了，所以基本上不可能再出什么花样，[具体参考](https://www.tr0y.wang/2022/06/06/SecMap-unserialize-pyyaml)，poc和ssti很类似

```python
常见poc
!!python/object/apply:subprocess.check_output [[calc.exe]]
!!python/object/apply:subprocess.check_output ["calc.exe"]
!!python/object/apply:subprocess.check_output [["calc.exe"]]
!!python/object/apply:os.system ["calc.exe"]
!!python/object/new:subprocess.check_output [["calc.exe"]]
!!python/object/new:os.system ["calc.exe"]
```

![image-20220827233029620](C:\Users\25952\AppData\Roaming\Typora\typora-user-images\image-20220827233029620.png)

本题的利用点就是这(原题由于版本低还是**yaml.load**，在新版中要利用需要加上unsafe)
这时候看看**yamlDatas**怎么来的？

![image-20220827233235752](C:\Users\25952\AppData\Roaming\Typora\typora-user-images\image-20220827233235752.png)

读的是文件**md5(filename).yaml**，这个**filename**就值得注意了，他是解压为rar文件前的yaml文件名

> 提一嘴，这里上传rar文件是因为在upload里用到了**unrar**，而且解压到的路径是**session里的路径加上/rar文件名**
> 唯一想不明白的就是为什么这里会产生一个这样的文件(文件名是md5(rar文件名))
> ![image-20220827234534641](C:\Users\25952\AppData\Roaming\Typora\typora-user-images\image-20220827234534641.png)
>
> 哦，破案了，在upload里面写入的
> ![image-20220827234820618](C:\Users\25952\AppData\Roaming\Typora\typora-user-images\image-20220827234820618.png)

接着上面的思路，既然他load的文件的文件名是可控的，有没有一种办法把有这个文件名的文件内容改了呢？？？？
够明显了吧，文件覆盖！！

所以现在的思路就是

> 将一个1.yaml压缩为.zip，控制文件解压路径为/fileinfo上传，得到**c4f6652bbeeb0c3244c43a522f517b76.yaml**
>
> 将一个**c4f6652bbeeb0c3244c43a522f517b76.yaml**压缩为.zip，控制文件解压路径为/fileinfo上传，成功覆盖
>
> 然后你就会发现**.zip**传过就不能传了

接下来有两种方式

第一种是不了解的话就完全不知道，**.rar1**也是可以被unrar解压的，所以上传.rar1就好了

**第二种**

在上面我们得知，其实你正常传，默认是会产生一个**md5(rar文件名).yaml**的，那这样也有了覆盖的办法

> 先上传一个1.rar，默认路径就行，此时会在fileinfo得到解压的文件**0611507e222914a26f498c68acdda9fc.yaml**
>
> 在用以**0611507e222914a26f498c68acdda9fc.yaml**压缩的.rar上传至fileinfo，然后
>
> ![image-20220828000158267](C:\Users\25952\AppData\Roaming\Typora\typora-user-images\image-20220828000158267.png)
>
> 成功！

此时访问**/display?file=1.rar**(第一次上传的rar文件)触发**yaml.load**

![image-20220828000342516](C:\Users\25952\AppData\Roaming\Typora\typora-user-images\image-20220828000342516.png)

反弹shell！

后面还有提权，本地是没办法复现了

参考博客:
https://www.cnblogs.com/kagari/p/16630947.html
https://pysnow.cn/archives/349/
**ghost1032YYDS**

