---
layout: post
title: "用Markdown编写Evernote文章(With Sublime Text)"
description: ""
category: "Tools"
---
{% include JB/setup %}

### 前言

先说说本文的来历。 之前用postach.io的服务构建了一个自己的个人博客，我也在陆续往这个博客上post文章。简单的来说，通过这个服务，你可以再Evernote的某一个笔记本里发布一篇文章，只要添加上特定的标签，这篇文章就可以自动发布到你的博客站点中。个人觉得这种方式有很多优点：

  * 发博文方便，你不需要像传统的方式那样在本地编号再登录到博客网站重新发布排版
  * 便于归档，你不用把文章存好几份，不用担心线上线下的文章数量是否一致
  * 有足够的定制空间。Postach.io可以从模板库里选择模板，也可以从github上fork一个现成的模板，修改一下应用到自己的博客上--这也是我现在正在使用的方式，我为自己的博客添加了评论系统，分享按钮，流量统计功能，等等......

<!-- more -->

但是缺点也是有的：

  * *只能用Evernote的编辑器去编写文章，说实话evernote的编辑器功能并不好，排版非常麻烦*

要说这个排版，最方便最"正确"的方式莫过于Markdown了，正好最近发现有一个Sublime Text 3的插件正好是用来发布Evernote的，如果能用Sublime编写Markdown形式的内容，然后简简单单直接post到Evernote上同时同步到postach.io上的话那可就真的完美了。经过一番探究，这种方法确实是可行的，于是在这里记录一下自己的经验做抛砖引玉之用。

* * *

### 配置篇

#### 第一步：为Sublime Text 3安装Evernote插件

安装Evernote插件的方法有三种，*Package Control*，*手动*，*Github*。这里根据我的经验，用*Github*和*手动*的方式真心容易出问题（我碰到了菜单不全的问题，貌似Github上的那个库是很久以前的版本，已经很久没更新了，只有发布新文章的功能），用*Package Control*的方法是最给力的。

*Package Control*是Sublime Text的一个插件，这个插件可以很方便的让Sublime去搜索安装其他插件，类似于一个包管理器，有了这个东西，以后安装更多的插件就非常容易了。所以首先我们得位Sublime Text装上Package Control：

> 安装方法来源：[一些必不可少的Sublime Text 2插件](http://blog.csdn.net/nivana999/article/details/7823805)
> 
>   1. 按Ctrl+`调出console
>   2. 粘贴以下代码到底部命令行并回车： import urllib2,os;pf='Package Control.sublime-package';ipp=sublime.installed_packages_path();os.makedirs(ipp) if not os.path.exists(ipp) else None;open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read())
>   3. 重启Sublime Text 2。
>   4. 如果在Perferences->package settings中看到package control这一项，则安装成功。

根据我的经验，照做就好，不会出问题的。

![](http://dellyqiao.qiniudn.com/2015/03/14/1.png/scale)


**更新：**

Mac版ST3按照上面ST2的方法安装成功，然而Windows版ST3无法成功。如果有问题请参照官方说明，ST3跟ST2的Package Control的安装方法应该是不同的。

<https://packagecontrol.io/installation#st3>

  


安装好之后，就可以去搜索安装Evernote插件了。（**注意：根据官方的说法，Evernote这个插件只兼容Sublime Text 3，目前3是Beta版，如果你的电脑里不是3，请去官网下载。**）

  1. 打开Sublime编辑器，调出命令面板。快捷键`CMD+Shift+P`
  2. 输入`Package Control Install`并回车。稍微等待一会儿，Sublime将会去搜索所有被Package Control管理的插件并且列出在编辑器列表中。
  3. 等列表出现之后，继续输入`evernote`，找到这个插件包之后，按下回车键开始安装。等到安装日志出现后就说明插件装好了。

	![](http://dellyqiao.qiniudn.com/2015/03/14/2.png/scale)


  4. 验证：CMD+Shift+P调出命令面板，输入evernote，看看有没有出现evernote相关的菜单，如果出现那就没问题了。

#### 第二步：为你的Sublime Text 3添加Evernote的授权

通过Evernote developer的授权，你就可以通过认证码进行文章的增删改查，做法如下：

  1. CMD+Shift+P调出命令面板（最喜欢这个强大的功能了ʅ(‾◡◝)），输入`evernote reconfigure auth`(**Sublime编辑器支持模糊匹配，所以不一定非得输入完整的命令，只要差不多就行），看到重新授权的菜单后回车。  

	![](http://dellyqiao.qiniudn.com/2015/03/14/3.png/scale)


  2. 这时候会自动弹出一个evernote的网页，网页的下部有一个创建token的连接，点击之，就会生成token了。  

![](http://dellyqiao.qiniudn.com/2015/03/14/4.png/scale)  

  3. 打开Sublime Text的界面，会看到在软件的下部出现了一个输入token的条，把token复制粘贴进去，回车；然后复制“NoteStore URL”，粘贴到软件中，回车。  

![](http://dellyqiao.qiniudn.com/2015/03/14/5.png/scale) 


不出意外的话，Sublime编辑器的evernote插件就完全配置好了。

* * *

### 使用篇

到使用的部分了，自然就是写文章+发布文章了。

#### 写文章

用`CMD+N`在Sublime中创建新文档并且通过`CMD+Shift+P`节日`ssmarkdown`设置Markdown语法匹配环境。

文章的标题和标签可以通过两种方式设定：

  1. 通过Liquid格式的meta data设置evernote文章的笔记本，标签和标题： 
  
	  	--- 
	  	title: 用Markdown编写Evernote文章(Sublime Text Editor) 	
	  	tags: ["published", "technology"] 
	  	notebook: dellyqiao.com >
	  	---  

如果meta data的数据格式不对的话，在发布的时候会报错。下图是我写本文时候的样子。  

	![](http://dellyqiao.qiniudn.com/2015/03/14/6.png/scale)

  2. 不设定Meta data，直接写文章，在发布的时候插件会提示你去设置标题，标签和所属的笔记本。

#### 发布文章

发布文章的命令是`send to Evernote as new note`，只需要几秒钟，你的文章就能按照想象的排版方式输出到evernote中并且同步到自己的博客里了。

当然，也完全可以打开现有的文章（`Open Evernote Note`）然后编辑保存，evernote的文章会按照原先的格式转换成Markdown格式输出到本地的Sublime Text中，修改完成后调用`Update Evernote Note`就OK了。

得益于Sublime Text强大的模糊搜索功能和各种各样的文档编辑功能，我觉得以后也许我会全靠Sublime Text来管理我的Evernote笔记了。

* * *

### 遗憾

任何事情达成的过程都是一个取舍的过程，这种方案也不例外，那就是图片的问题。

用Sublime打开带图片的Evernote文章的时候，图片不会被正确的转换成Markdown格式

  * 如果图片是直接复制粘贴到Evernote笔记里的，转换成Markdown的时候图片会转换成一堆无法识别的html代码。如果不加任何修改再把那堆代码同步到evernote中，evernote也无法识别。
  * 如果图片是在用Markdown编写文章的时候通过`![alt "text"]()`标签插入的，发布到evernote再转出来的话会转换成几个特殊字符，也不是正常的图片。

个人觉得靠谱的解决方案如下：

  * 用第三方的网盘存储图片，然后生成公开连接，在写文章时通过标签引用。如下： `![](http://dellyqiao.qiniudn.com/2015/03/14/2.png/scale)` （同时也裂墙推荐七牛云存储，免费好用）

	如此一来也能有效地节省Evernote吝啬的60M上传流量吧。如果这篇文章以后有可能会有大量改动的话，而且确实非Markdown编辑器不用的话，把Markdown文件在本地保存一份，方便随时修改而不用再去头疼图片的问题。

  * （*从来不用Evernote客户端一直用网页端的请无视，网页端的复杂度堪比图片外链。。。*）用Sublime写文章时只写文字，等文字发布之后，再转到Evernote客户端中把需要的图片粘到文章里做二次修改。 这种情况适合以后改动较少和图片较多的情况（按我的理解，在evernote客户端里粘贴图片是一种更为简单方便的操作）。

* * *

文章写到这儿也算完成了，我觉得介绍的还算详细，有了这工具，修改整理自己的笔记真心也方便了不少，很容易就能做出良好排版的笔记。如果我需要的话，可以把笔记转换到本地，然后用Markdown语法稍加修改，然后生成格式良好的打印版文档。

抛砖引玉，Evernote的客户端编辑器确实做的不咋地，用Markdown+Sublime Text 3，相信你更会感受到创造内容的快乐的。
