## php特性

`strstr`可以用大小写绕过

`str_replace`也可以用大小写绕过

`stristr`函数对大小写不敏感

--XCTF Web_php_include

+++

`*`可代替很多字符` ?`只能代替一个  exp:`cat f\*`==`cat fla?.php`

+++

**常见空字符**

| 原值            | URL(ascii) |
| --------------- | ---------- |
| 截断符          | %00        |
| 空格            | %20        |
| tab(水平制表符) | %09        |
| tab(垂直制表符) | %0b        |
| 换页符          | %0c        |
| 换行符          | %0a        |
| 回车符          | %0d        |

+++

> 在实战中文件包含漏洞配合PHP的伪协议可以发挥重大的作用,比如读取文件源码,任意命令执行或者开启后门获取webshell等,常用的伪协议有
>
> php://filter 读取文件源码
> php://input 任意代码执行
> data://text/plain 任意代码执行
> zip:// 配合文件上传开启后门

+++

## xdebug

sb php xdebug，一辈子被人瞧不起😑！

phpstorm整了一早上没整好，还得是vscode

**xdebug2**

```ini
[xdebug]
zend_extension=E:/phpstudy_pro/Extensions/php/php7.3.9nts/ext/php_xdebug.dll
xdebug.remote_enable = On
xdebug.remote_handler = dbgp
xdebug.remote_host= localhost
xdebug.remote_port = 9003
xdebug.idekey = PHPSTORM
```

**xdebug3**

```ini
[Xdebug]
zend_extension=E:/phpstudy_pro/Extensions/php/php8.0.2nts/ext/php_xdebug.dll
xdebug.mode=debug
xdebug.start_with_request = yes
```

**.vscode**

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003
        },
        {
            "name": "Launch currently open script",
            "type": "php",
            "request": "launch",
            "program": "${file}",
            "cwd": "${fileDirname}",
            "port": 0,
            "runtimeArgs": [
                "-dxdebug.start_with_request=yes"
            ],
            "env": {
                "XDEBUG_MODE": "debug,develop",
                "XDEBUG_CONFIG": "client_port=${port}"
            }
        },
        {
            "name": "Launch Built-in web server",
            "type": "php",
            "request": "launch",
            "runtimeArgs": [
                "-dxdebug.mode=debug",
                "-dxdebug.start_with_request=yes",
                "-S",
                "localhost:0"
            ],
            "program": "",
            "cwd": "${workspaceRoot}",
            "port": 9003,
            "serverReadyAction": {
                "pattern": "Development Server \\(http://localhost:([0-9]+)\\) started",
                "uriFormat": "http://localhost:%s",
                "action": "openExternally"
            }
        }
    ]
}
```

