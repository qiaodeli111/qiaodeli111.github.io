---
layout: post
title: "设置IIS服务器监听特定IP的特定端口（IIS与IHS或Apache共存的解决方案）"
description: ""
category: "Middleware"
---
{% include JB/setup %}


在某些场景中，需要IIS与Apache类的中间件产品共存的情况。如果你的服务器换过网卡，也许会出现跟本文类似的IIS无法启动的情况。如果碰到了同样的问题，可以参考本文去修复。

<!-- more -->

IIS启动时错误弹窗，信息如下：

	"The network location cannot be reached, xxxxxx"

Windows报的这种弹窗错误，也只能通过Windows自带的事件管理器里查看了。直接运行：eventvwr，打开SYSTEM项，看到新地址没有被监听的错误，服务器无法接受来自这个IP的请求。


### 问题分析

多方搜集资料后发现，如果是首次配置IIS，那么用IIS的图形界面工具添加绑定就没问题了。

但是如果换过网卡，仅通过图形界面无法让IIS去监听某一个IP地址的d某个端口，必须要用命令行`httpcfg`工具来把监听列表更新一下才可以。猜想这个监听列表是属于系统文件的一部分，IIS在启动的时候可能是会依据这个文件作为更高优先级别的端口绑定。


按照如下步骤进行配置：


1. `httpcfg query iplisten`

列出现在的监听列表。

2. `httpcfg delete iplisten -i 10.50.165.45`

移除旧的监听IP地址。

3. `httpcfg set iplisten -i 10.52.21.125`

添加新的IP地址到监听列表。


再次测试依然失败。。。为什么？重启一下Server就好了。。。。。。(注意是重启服务器，重启IIS服务不会起作用的）

Server重启之后查看一下监听的端口和IP：

	netstat -ano | findstr 10.52.21.125

可以看到进程4监听10.52.21.125的80和443端口。进程4是一个系统进程，IIS貌似也是属于挂在这个进程里边的一个子进程（猜想。。。不太懂IIS）。

好了IIS设置好了，接下来配置Apache的配置文件，让virtual host的IP不同于IIS（比如设置为10.52.21.45），同样也可以正常启动。


所以现在这台服务器既可以接受向IP：10.52.21.125发送的请求（IIS会处理这些请求），也可以接受向IP：10.52.21.45发送的请求（IHS会处理这些请求）。
