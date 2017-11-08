---
title: git相关操作
date: 2017-08-02 22:40:20
tags: [git]
category: "开发工具"
---
### 一. git关联本地与远程分支

```
$ git branch --set-upstream-to=origin/dev2.2 master
Branch master set up to track remote branch dev2.2 from origin.
```
远程分支在前，本地分支在后。
关联之后就可以正常的pull代码了。

### 二.查看所有分支（本地、远程）
```
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/dev
  remotes/origin/dev2.2
  remotes/origin/master
```

### 三.当前关联的远程分支
```
$ git branch -vv
* master beb7725 [origin/dev2.2: behind 1] 首页本地记录上次关闭之前的状态
```

### 四.git merge 和 git merge --no-ff
![Alt text](http://img.blog.csdn.net/20150811134840627 "Optional title")
根据这张图片可以看出
Git merge –no-ff 可以保存你之前的分支历史。能够更好的查看 merge历史，以及branch 状态。
git merge 则不会显示 feature，只保留单条分支记录。

### 五.merge 与 rebase 的区别
1.Git merge 会生成一个新得合并节点，而rebase不会
比如：
```
      D---E test
     /
A---B---C---F master
```
使用merge合并, 为分支合并自动识别出最佳的同源合并点： git merge master
```
      D--------E
     /          \
A---B---C---F----G   test, master
```
操作会舍弃 master 分支上提取的 commit，同时不会像 merge 一样生成一个合并修改内容的 commit，相当于把 master 分支（当前所在分支）上的修改在 deve 分支（目标分支）上原样复制了一遍:git rebase test
```
A---B---D---E---C'---F'   test, master
```
使用git pull时默认是merge， 加 --rebase参数使其使用rebase方式
```
git pull --rebase  
```
2.rebase和merge的不同适用场景

**rebase场景：**如果修改了某个公用代码的BUG，这个时候就应该是把所有的OEM版本分支rebase到这个修复BUG的分支上来，在rebase过程中，Git会要你手动解决代码上的冲突，你需要做的就是把修复BUG的代码放到目标分支代码里面去。rebase的结果是：所有的分支依然存在

**merge场景：**因为成员的代码开发工作已经完成了，也不需要再保留这个分支了，所以我们可以把这个成员分支merge到主分支上，当然冲突在所难免，手工解决的工作肯定逃不掉，但是利大于弊不是吗。merge以后，分支就不存在了，但是在git的所有分支历史中还能看到身影。

**一般我们把别的分支合并到master时用merge，而把master合并到别的分支时会用到rebase的原因，这是因为master分支一般commit会比较频繁。**

所以每次下拉代码fetch之后用rebase的原因就是： 
本地commit之后，fetch远端代码，此时，远端代码可能会被若干人修改会有若干个commit，而本地就一个commit，然后git rebase的时候，是默认rebase 远端代码，此时会将本地commit应用到远端代码，也就只需要解决一次冲突，并且rebase之后没有新的commit，很友好。但是，如果使用merge，则会产生新的commit。

### 六.网上推荐的工作流一般是用fetch+rebase (相比pull+merge工作流更干净，不容易出错)
**比如dev是你的公共开发分支***

git checkout dev  # 本地切到公共分支 

git pull              # 将本地的dev更新

git checkout -b bug_101026  # 新建一个主题分支（一个bug，一个功能什么的）
... # 改动.. commit.. 测试...

git fetch origin      # 更新upstream

git rebase origin/dev  # 将你的commits移到的末尾

git checkout dev  # 切换到公共分支

git pull              # 更新公共分支

git rebase bug_101026  # 将你的主题分支加到公共分支的末尾

git push               # 推送
