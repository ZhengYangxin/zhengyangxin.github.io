---
title: Subline3下的两款Markdown插件
date: 2017-03-18 22:38:01
category: 工具 
tags: [Subline,Markdown]
---
### MarkdownEditing
#### MarkdownEditing是Markdown写作者必备的插件，它可以不仅可以高亮显示Markdown语法还支持很多编程语言的语法高亮显示。
1. 安装插件
安装插件之前，我们需要首先安装一个Sublime 中最不可缺少的插件 Package Control, 以后我们安装和管理插件都需要这个插件的帮助。
2. 安装"Package Control"
使用快捷键 " ctrl + `" 打开Sublime的控制台 ,或者选择 View > Show Console 
在控制台的命令行输入框，把下面一段代码粘贴进去，回车 就可以完成Pacakge Control 的安装了。
```
import urllib.request,os,hashlib; h = 'eb2297e1a458f27d836c04bb0cbaf282' + 'd0e7a3098092775ccb37ca9d6b2e4b7d'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```
3. 安装MarkdownEditing
Package Control 安装成功后我们就可以使用它方便的管理插件了，首先使用快捷键 'command + shift + p ' 进入到Sublime 命令面板，输入 "package install" 从列表中选择 "install Package" 然后回车。这时候Sublime开始请求远程插件仓库的索引，所以第一次使用可能会有一些小的延时。
![Alt text](http://upload-images.jianshu.io/upload_images/222358-bee1d25963b27ea7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "Optional title")
看到列表的更新之后输入 "markdown ed" 关键字，选择“MarkdownEditing" 回车。 插件安装完毕后需要重新启动Sublime插件才能生效。下面是我使用sublime编辑代码片断的显示效
![Alt text](http://upload-images.jianshu.io/upload_images/222358-d8421f8682fdd2a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "Optional title")

输入 "mdi + tab" 会自动插入下面的图片标记
```
![Alt text](/path/to/img.jpg "Optional title")
```
输入 "mdl + tab" 会自动生成下面的链接标记
```
[](link)
```
### Markdown Preview 插件
#### Mardown Preview不仅支持在浏览器中预览markdown文件，还可以导出html代码。
 1. 安装
通过按组合键Ctrl+Shift+P或是点击Preference->Package Control调出命令面板，然后再输入 install，选择 Package Control: install package。 
![Alt text](http://img.blog.csdn.net/20160424184816763 "Optional title")
在插件安装面板输入markdown找到Markdown Preview并点击安装即可。
![Alt text](http://img.blog.csdn.net/20160424211042612 "Optional title")
 2. 使用
通过按组合键Ctrl+Shift+P或是点击Preference->Package Control调出命令面板，输入mdp，下图中红框圈出的就是在浏览器中预览markdown文件。 
![Alt text](http://img.blog.csdn.net/20160424211409258 "Optional title")
选中后，你将见到两个选项：GitHub和Mardown。GitHub选项意味着使用GitHub的在线API来解析.md文件。它的解析速度取决于你的联网速度。据称有每天60次访问的限制。[2]但能免费获得GFM格式的语法支持和EMOJI表情的支持。
另外一个常用功能是图中第五个，Export HTML in Sublime Text，即导出html文件到sublime text。
 3. 快捷键设置
Sublime Text支持自定义快捷键，markdown preview默认没有快捷键，我们可以自己为preview in browser设置快捷键。方法是在Preferences -> Key Bindings User打开的文件的中括号中添加以下代码(可在Key Bindings Default找到格式)：

```
{ "keys": ["alt+m"], "command": "markdown_preview", "args": {"target": "browser", "parser":"markdown"}  }
```
###### 这里： 
"alt+m"可设置为自己喜欢的按键。 
"parser":"markdown"也可设置为"parser":"github"，改为使用Github在线API解析markdown。
