---
layout: post
title: "Configuring the servers to start automatically at boot"
description: ""
category: ["security", "middleware"]
---
{% include JB/setup %}


## 如何配置WAS服务跟随系统启动 ##

默认情况下，WAS应用服务器，NodeAgent等等WAS服务是不会跟随系统自动启动的。本文将介绍如何配置WAS使其服务跟随系统启动。

WAS自动启动有两种方法，首先介绍第一种方法：

### 配置WAS服务器跟随NodeAgent/Node自动启动 ###

1. 登录到管理控制台。
2. 跳转到Servers > Server Types > WebSphere application servers。
3. 找到希望自启动的服务器，跳转到其配置界面，打开Java and Process Management，点击Monitoring Policy，如下图：

<!-- more -->
	
	![http://dellyqiao.qiniudn.com/2016%2F05%2F10%2Fautostart_1.png](http://dellyqiao.qiniudn.com/2016%2F05%2F10%2Fautostart_1.png/scale)

4. 勾选Automatic restart，并且配置“Node restart state”为“RUNNING”。如图：

	![http://dellyqiao.qiniudn.com/2016%2F05%2F10%2Fautostart_2.png](http://dellyqiao.qiniudn.com/2016%2F05%2F10%2Fautostart_2.png/scale)

这样当NodeAgent/Node启动的时候，应用服务器也会自动启动。但是很明显，想要让这种方法生效，前提是NodeAgent/Node必须可以跟随操作系统启动。所以接下来我们需要继续配置：

### 配置NodeAgent/Node跟随操作系统启动 ###

1. 打开命令提示符窗口（Windows下可以运行cmd，Linux下打开一个新的Shell窗口）。
2. 跳转到`<WAS Home>/bin`。

		例如：
		在Windows下：C:\IBM\WebSphere\AppServer\bin
		Linux下：/opt/IBM/WebSphere/AppServer/bin
		视实际安装目录而定。

3. 使用wasservice工具把NodeAgent添加到随系统启动的列表里：
	
		wasservice -add nodeagent -serverName nodeagent -profilePath “C:\IBM\WebSphere\AppServer\profiles\DocsWin01” -restart true -startType automatic

	结果如下图：
	
	![http://dellyqiao.qiniudn.com/2016%2F05%2F10%2Fautostart_4.png](http://dellyqiao.qiniudn.com/2016%2F05%2F10%2Fautostart_4.png/scale)

4. 如果以上命令失败，则可能是权限问题。可以尝试以管理员身份启动命令提示符窗口。

	![http://dellyqiao.qiniudn.com/2016%2F05%2F10%2Fautostart_3.png](http://dellyqiao.qiniudn.com/2016%2F05%2F10%2Fautostart_3.png/scale)
	
> 注意：
> 
> NodeAgent只存在于ND版本中，对于AdminAgent+APP Server的组合来说，要确保AdminAgent随系统自动启动。命令也是类似：
> 
> wasservice -add adminagent -serverName adminagent -profilePath “C:\IBM\WebSphere\AppServer\profiles\AdminAgent01” -restart true -startType automatic
		

**至此，所有的WAS服务器都可以做到随系统启动了。**


基于以上的经验，我们也很容易想到另外一种让WAS服务器自动启动的方法，那就是利用wasservice工具，把所有的WAS应用服务器也添加到自启动列表。这样做的好处就是应用服务器进程的自启动不依赖于Node的状态，即使Node出问题，管理控制台无法访问，应用服务器还是**可能**启动。

	示例如下：
	wasservice -add server1 -serverName server1 -profilePath “C:\IBM\WebSphere\AppServer\profiles\AppSrv01” -restart true -startType automatic


----------


全文完。两种方法大家自行选择。