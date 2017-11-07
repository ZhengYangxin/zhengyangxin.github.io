---
title: 阿里云Centos7搭建Git服务器仓库
date: 2017-02-09 20:30:45
tags:
---
阿里云Centos7 搭建Git服务器仓库，记录过程

## 1.首先需要安装Git，可以使用yum源在线安装：
```
[root@localhost Desktop]# yum install -y git
```
![这里写图片描述](http://img.blog.csdn.net/20170209181003136?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWFuZ3hpbjExMTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
## 2.创建一个git用户，用来运行git服务
```
# adduser git  
```
![这里写图片描述](http://img.blog.csdn.net/20170209181024793?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWFuZ3hpbjExMTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
## 3.初始化git仓库：这里我们选择/usr/local/tomcat7/webapps/baby_android/来作为我们的git仓库
```
# git init
```
仓库路径  /usr/local/tomcat7/webapps/baby_android/

![这里写图片描述](http://img.blog.csdn.net/20170209181915113?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWFuZ3hpbjExMTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
执行以上命令，会创建一个裸仓库，裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以.git结尾。然后，设置权限：
```
[root@localhost webapps]# chown - Rh git:users baby 
```

## 5.创建SSH Key
 首先在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有id_rsa和id_rsa.pub这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：
```
$ ssh-keygen -t rsa -C "youremail@example.com"
```
你需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码。
如果一切顺利的话，可以在用户主目录里找到.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。

## 6.Git服务器打开RSA认证
然后就可以去Git服务器上添加你的公钥用来验证你的信息了。在Git服务器上首先需要将/etc/ssh/sshd_config中将RSA认证打开，即：
```
1.RSAAuthentication yes    
2.PubkeyAuthentication yes    
3.AuthorizedKeysFile  .ssh/authorized_keys
```

这里我们可以看到公钥存放在.ssh/authorized_keys文件中。所以我们在/home/git下创建.ssh目录，然后创建authorized_keys文件，并将刚生成的公钥导入进去。
创建文件夹 mkdir 路径/文件夹名
创建文件 vi 路径/文件名
然后再次clone的时候，或者是之后push的时候，就不需要再输入密码了：

##  7.禁用git用户的shell登陆
出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑/etc/passwd文件完成。找到类似下面的一行：
```
git:x:1001:1001:,,,:/home/git:/bin/bash 
```
最后一个冒号后改为：
![这里写图片描述](http://img.blog.csdn.net/20170209185636959?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWFuZ3hpbjExMTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
```
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell  
```
 
 git clone git@42.96.150.57: /usr/local/tomcat7/webapps/baby/

## 8. 客户端基本操作
git add .  
git commit
git push

## 9.服务器自动更新部署
1. 进入到  /usr/local/tomcat7/webapps/baby/.git 文件夹中，会发现 .git/hook 文件夹在里面，进入到 hook 中，里面有很多的 sample 脚本，这里我们只需要用到 post-update。
post-update脚本
2. 设置文件权限
``` 
chown -Rh git:users baby
```

 
## 10.创建git服务器远程仓库

```
$ mv post-update.sample post-update
    $ vim post-update
```


## 注意：
1. Git: push 出错的解决 master -> master (branch is currently checked out)
这是由于git默认拒绝了push操作，需要进行设置，修改.git/config添加如下代码：
```
    [receive]
    denyCurrentBranch = ignore
```
线上添加文件设置权限
``` 
chown -Rh git:users baby
```
