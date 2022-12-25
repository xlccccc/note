### [buu]ningen

+++

使用binwalk发现有zip

```sh
xiaolei@xl-pc:~/cloacked-pixel/images$ binwalk 9e3ec8c2-38c7-41cf-b5d7-abe7872de4c3.jpg

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
38689         0x9721          Zip archive data, encrypted at least v2.0 to extract, compressed size: 50, uncompressed size: 38, name: ningen.txt
38871         0x97D7          End of Zip archive, footer length: 22
```

有提示得知是四位数数组密码，使用`fcrackzip`得到密码**8368**

![image-20220807112230768](D:\Typora\note\CTF\misc\challenge.assets\image-20220807112230768.png)

```shell
xiaolei@xl-pc:~/cloacked-pixel/images/_9e3ec8c2-38c7-41cf-b5d7-abe7872de4c3.jpg.extracted$ fcrackzip -b -c '1' -l 4 -u 9
721.zip


PASSWORD FOUND!!!!: pw == 8368
```

+++

### [buu]小明的保险箱

+++

与上题基本相同，不同的是这是rar文件，需要安装rar命令来解压，破解也不能用fcrackzip了，直接在win上破解吧

`rar x 1381F.rar`

+++

### [buu]爱因斯坦

+++

与上面相同，密码在属性的备注里面

+++

### [buu]easycap

+++

用wireshark打开，全是TCP流，追踪TCP流得到flag
![image-20220807114517789](D:\Typora\note\CTF\misc\challenge.assets\image-20220807114517789.png)

+++

### [buu]FLAG

+++

一张图片，用010打开没有什么发现
然后丢进**StegSolve**，lsb之后发现是**PK**开头

另存为zip然后直接压缩打不开，丢进010看估计是因为有很多无用数据
但用**7z**顺利解压了

发现是一个ELF开头的文件，直接放入linux执行得到flag

![image-20220906233723217](D:\Typora\note\CTF\misc\challenge.assets\image-20220906233723217.png)

+++

### [buu]神秘龙卷风

+++

**ARCHPR**破解出密码 5463

然后得到一个这种东西

![image-20220906234410653](D:\Typora\note\CTF\misc\challenge.assets\image-20220906234410653.png)

查阅后得知是**Brainfuck**代码

![img](D:\Typora\note\CTF\misc\challenge.assets\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzU3MDYxNTEx,size_16,color_FFFFFF,t_70.png)

[bf在线运行](http://bf.doleczek.pl/)，网站在线运行一下就行

+++



