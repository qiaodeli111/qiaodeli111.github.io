﻿---
layout: post
title: "在vim中配置Markdown Preview插件"
description: ""
category: ["Tools"]
---
{% include JB/setup %}

Vim可以算是我的主力文本编辑器，这东西上手之后是一个非常好用的工具。Vim适合写市面上大部分的开发语言和脚本，当然用来写Markdown也是非常合适的，或者说越来越合适了。

相比较其他五花八门的Markdown编辑器，他们的书写体验都不是很好，基本上就是一个记事本加上了预览功能，但是他们共同比Vim强的一点就是：支持实时预览。今天我就要为Vim加上实时预览的功能。

首先看一下效果，非常漂亮。感谢此开源项目的作者iamcco。

![MarkdownPreview](https://cloud.githubusercontent.com/assets/5492542/15363504/839753be-1d4b-11e6-9ac8-def4d7122e8d.gif)

> 本文以Win10环境为例，Linux/Mac用户请根据实际情况调整其中的参数


----

### 第一部分：安装[junegunn/vim-plug](https://github.com/junegunn/vim-plug)。

> 这是一个用来管理Vim插件的一个插件。直接把github上的项目名称配置进去，这个插件就可以通过一条命令直接下载项目，并且安装到Vim里。如果插件有更新，也可以通过一条命令直接更新全部插件，非常好用。我们用到的这个Markdown插件也是通过这个vim-plug来安装的。

<!-- more -->

首先需要安装git。[点击这里](https://git-scm.com/downloads)获取下载连接。
  
vim-plug的安装过程也非常方便，主要分两步。

* 下载github项目，放置到Vim的插件目录。这一部分作者已经给出了一套自动脚本，如下：

`Unix`

	curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
	https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

`Windows (PowerShell)`

	md ~\vimfiles\autoload
	$uri = 'https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
	(New-Object Net.WebClient).DownloadFile(
	  $uri,
	  $ExecutionContext.SessionState.Path.GetUnresolvedProviderPathFromPSPath(
	    "~\vimfiles\autoload\plug.vim"
	  )
	)

**需要注意的是Vim配置文件和安装路径要根据自己的实际情况来，Linux一般不需要多作更改，Win10的话可以把路径改为绝对路径或者相对路径。**


* 修改配置文件。把下面的内容拷贝到Vim配置文件的最上面。Linux默认为用户根目录下的`.vimrc`，Win10默认为安装路径下的`_vimrc`。

  	 
	" Plugins will be downloaded under the specified directory.
	call plug#begin('D:\ProgramFiles\Vim\vimfiles\plugged')

	" Declare the list of plugins.
	Plug 'tpope/vim-sensible'
	Plug 'junegunn/seoul256.vim'

	" List ends here. Plugins become visible to Vim after this call.
	call plug#end()

* **重启Vim**，输入`:PlugInstall`，上文中配置的两个插件就会自动下载安装了。将来如果需要安装新的插件，只需要把`Plug xxxx/xxxx`配置到Vim配置文件里并且重复执行`:PlugInstall`就好了。


----

### 第二部分：安装[iamcco/markdown-preview.vim](https://github.com/iamcco/markdown-preview.vim).

基于vim-plug已经成功安装，我们这个插件的安装就会比较容易。只需要把下面两行配置放到vim配置文件里，然后重启Vim，运行`:PlugInstall`即可。

	Plug 'iamcco/mathjax-support-for-mkdp'
	Plug 'iamcco/markdown-preview.vim'

插件安装完成后，再把下面四行配置放到vim配置文件中（可以放到最后面），用来映射快捷键：

	nmap <silent> <F8> <Plug>MarkdownPreview        " for normal mode
	imap <silent> <F8> <Plug>MarkdownPreview        " for insert mode
	nmap <silent> <F9> <Plug>StopMarkdownPreview    " for normal mode
	imap <silent> <F9> <Plug>StopMarkdownPreview    " for insert mode

这些配置的意思是说，在普通模式下（n）和插入模式下（i），按下F8键就可以实时预览，按下F9键就可以停止预览。


----

好了，安装完成，可以享受在Vim里编辑Markdown同时享受实时预览的快感了
。
。
。
。
。
。
。
。
。
。
。
。
。
。
。
。
。
吗？

才不会呢。。

这个Markdown-preview插件是需要Vim编辑器有Python2或者3的支持的。通过`vim --version`可以查看你安装的Vim支持哪些功能，不支持哪些功能。

对于Linux版本来说，如果你的Vim不支持Python，请重新编译安装，我感觉这可能是唯一的解决办法。具体方法看这里（未亲测）[Vim 8.0 版本安装方法及添加Python支持](https://www.cnblogs.com/DillGao/p/6268165.html)。

对于Windows版本来说，Windows用户是不太可能编译安装并且修改编译时参数的。Vim安装号之后，会提供两个版本，一个是非图形界面的Vim.exe，另一个是图形界面的gvim.exe。两个应用对Python的支持居然是不一样的（笑）。

具体是不是这样呢，还是通过`vim --version`和`gvim --version`去查看。

即便gvim是支持原生支持Python的，但是我安装Python的时候并没有安装在默认路径下，所以我们还需要在Vim配置文件里制定Python库的位置，比如我的：

	set pythonthreedll=C:\Users\qiaod\AppData\Local\Programs\Python\Python36-32\python36.dll

修改完之后，再次重启Vim，输入`:python print("Hello World!")`用来验证Python配置是否生效。如果已生效，打开md文件之后就可以按F8直接实时预览了。


----


这次是真行了。

![preview](http://dellyqiao.qiniudn.com/2018/04/10/preview.JPG)
