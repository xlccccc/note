## git connect github by ssh

### process

```sh
#首先本地生成密钥
xiaolei@xl-pc:~/learn/learnGit/gittest$ ssh-keygen -b 4096 -t rsa -C "2595251998@qq.com"
#生成后将该密钥加入到你的ssh中
xiaolei@xl-pc:~/learn/learnGit/gittest$ ssh-add ~/.ssh/id_rsa
#正常返回则成功
xiaolei@xl-pc:~/learn/learnGit/gittest$ssh git@github.com  
#此时git push仍然需要账号密码
xiaolei@xl-pc:~/learn/learnGit/gittest$ git config --get remote.origin.url
#若返回的是https协议的 说明此时是通过账号密码或者token来连接git的
xiaolei@xl-pc:~/learn/learnGit/gittest$ git remote set-url origin git@github.com:xlccccc/xlccccc-firsthtml.git
#改为git协议，可以git push
#所以以后下载远程仓库时使用git协议而非https协议
xiaolei@xl-pc:~/learn/learnGit/gittest$ git clone git@github.com:xlccccc/xlccccc-firsthtml.git
#windows主机为什么可以呢？疑惑
```

### tips

```sh
#git一般连不上
#clash打开局域网共享，然后
xiaolei@xl-pc:~/learn/learnGit/gittest$ export ALL_PROXY="http://192.168.31.165:7890"
```

只能保证当前shell是走代理的

之前clash局域网共享一直用不了是因为**clash-win64**规则未启用

## multiple repository

```sh
#添加新分支
xiaolei@xl-pc:~/learn/learnGit/gittest$ git remote add skeleton https://github.com/Berkeley-CS61B/skeleton-sp21.git
#可以看到多了两个分支
xiaolei@xl-pc:~/learn/learnGit/gittest$ git remote -v
skeleton        https://github.com/Berkeley-CS61B/skeleton-sp21.git (fetch)
skeleton        https://github.com/Berkeley-CS61B/skeleton-sp21.git (push)
#更新分支内容
xiaolei@xl-pc:~/learn/learnGit/gittest$ git pull skeleton master
remote: Enumerating objects: 360, done.
remote: Total 360 (delta 0), reused 0 (delta 0), pack-reused 360
Receiving objects: 100% (360/360), 512.53 KiB | 489.00 KiB/s, done.
Resolving deltas: 100% (124/124), done.
From https://github.com/Berkeley-CS61B/skeleton-sp21
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> skeleton/master
#如果某文件想回到前版本(文件夹也行)
xiaolei@xl-pc:~/learn/learnGit/gittest$ git checkout uid -- proj0/test.txt
```





```
export ALL_PROXY='socks5://47.93.215.154:10000'
```













