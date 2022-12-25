## linux

+++

### 前言

+++

因受不了**wsl2 ubuntu22.04**的各种莫名其妙的bug，还是决定回到20版的
目前安装了
**python2**
**GitHack两个版本**
**wget**
**vim**
**screenfetch**
**ifconfig**
其它就是些无关紧要的东西

+++

### 卸载当前ubuntu

+++

首先注销掉当前的用户

```powershell
wslconfig /u Ubuntu-22.04
```

然后直接在设置里面卸载
卸载不了就

```powershell
wsl --unregister Ubuntu-22.04
```

这样就卸载完成了

+++

### 安装环境

+++

```sh
root@xl-pc:~# apt install screenfetch
root@xl-pc:~# apt install python2
xiaolei@xl-pc:~$ git clone https://github.com/lijiejie/GitHack
xiaolei@xl-pc:~$ git clone https://github.com/BugScanTeam/GitHack
#     xiaolei@xl-pc:~$ curl https://get.docker.com | sh  //安装docker
#     root@xl-pc:~# service docker start  //启动docker
```

+++

### 安装vulhub

+++

```sh
root@xl-pc:~# docker search vulhub
root@xl-pc:~# docker pull vulhub/weblogic  //拉取星级最高的
```

下载了半天，发现不应该是这样搭建vulhub
正好wsl2拉取的镜像在windows死活打不开，wsl2里面curl也curl不通(大吐血，浪费时间)
于是决定直接在windows里面安装docker了

+++

## windows

+++

### 安装docker

+++

直接在官网下载`Docker Desktop`，安装就行，很简单

+++

### 安装docker-compose

+++

也很简单

```powershell
PS C:\Users\25952> pip install docker-compose
```

+++

### 安装git

+++

同样也是官网下载`git安装包`
安装的时候有些默认配置需要改变，可以跟着教程来

+++

### 安装vulhub

+++

```powershell
PS C:\Users\25952> git clone https://github.com/vulhub/vulhub E:\resouses\git_ku\vulhub
#然后进入要复现的漏洞文件，例子/flask/ssti
PS E:\resouses\git_ku\vulhub\flask\ssti> docker-compose build
PS E:\resouses\git_ku\vulhub\flask\ssti> docker-compose up -d
#这样就开启了环境
PS E:\resouses\git_ku\vulhub\flask\ssti> docker-compose ps 
#看端口，然后直接 localhost:port 就可以进入靶场了
PS E:\resouses\git_ku\vulhub\flask\ssti> docker-compose down
#关闭靶场
```

+++

### 安装vulfocus

+++

先直接在docker pull了试试，但是弄了半天
先是镜像库里面没有同步镜像
后来又是下载不了镜像
是在没办法，看到用`docker-compose`搭建的例子，试了一下，基本正常
就是开启靶场的时候一直转圈
![image-20220723142716154](D:\Typora\note\linux\wsl2_remake.assets\image-20220723142716154.png)

但实际上docker容器里面已经有了
![image-20220723142743486](D:\Typora\note\linux\wsl2_remake.assets\image-20220723142743486.png)

这样不能提交flag了，虽然问题不大(事实上好像都没生成flag，不过问题也不大...)
只不过没有源码怎么做题？

+++