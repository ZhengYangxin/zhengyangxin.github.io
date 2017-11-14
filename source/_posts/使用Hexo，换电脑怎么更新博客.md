---
title: 使用Hexo，换电脑怎么更新博客
date: 2017-11-10 15:33:53
tags: "经验"
---
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=474567580&auto=1&height=66"></iframe>
hexo官方给了一些迁移的方法，不过它上面介绍的方法都是把博客文章从hexo系统迁移到其他博客系统的方法。然而我们这里要讨论的是：
#### 当我们更换电脑的时候我们应该怎么办？
所以默认你已经成功利用hexo和github发布博客，如果还没有，网上很多资料。
<!--more-->
具体的思路是：在生成的已经推到github上的hexo静态代码出建立一个分支，利用这个分支来管理自己hexo的源文件。如果能在刚刚配置hexo的时候就想好以后的迁移的问题就太好了，可以省掉很多麻烦，可实际使用中，刚刚配置hexo的时候，好多人都是初学，不会想到以后的问题，我就是这样的。

### 具体操作：
1. 克隆gitHub上面生成的静态文件到本地
```
git clone git@github.com:ZhengYangxin/zhengyangxin.github.io.git
```
2. 把克隆到本地的文件除了git的文件都删掉，找不到git的文件的话就当删了吧。不要用hexo init初始化。
3. 将之前使用hexo写博客时候的整个目录（所有文件）搬过来。把该忽略的文件忽略了
```
.DS_Store
Thumbs.db
db.json
*.log

public/
.deploy*/
node_modules
##注意忽略node_modules,这个文件换一台电脑都需要用npm install 重新生成
```
4. 创建一个叫hexo的分支
```
git checkout -b hexo
```
5. 提交复制过来的文件并推送至hexo分支
```
git add --all
git commit -m "新建分支源文件"
git push --set-upstream origin hexo
```

到这里基本上就搞定了，以后再推就可以直接git push了，hexo的操作跟以前一样。
今后无论什么时候想要在其他电脑上面用hexo写博客，就先装好node然后直接把创建的分支克隆下来，npm install安装依赖之后就可以用了。

```
git clone -b hexo git@github.com:ZhengYangxin/zhengyangxin.github.io.git
```

这样做完了以后，每次写完博客发布之后不要忘了还要git push把源文件推到分支上。