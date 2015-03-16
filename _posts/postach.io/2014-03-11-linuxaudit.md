---
layout: post
title: "Linux服务器的一些audit的方法"
description: ""
category: ["Server", "Security"]
---
{% include JB/setup %}


背景：

JBoss服务被重启，根据JBoss log output的说法，那个JBoss服务应该是被某人重启过，所以现在要找出到底是谁做的。（JBoss stopping Output可以参照这里：[https://developer.jboss.org/wiki/ShutdownLogs](https://developer.jboss.org/wiki/ShutdownLogs)


首先，找到最近连接过server的所有用户：

	last

输出结果类似这样的：

<!-- more -->

	reboot system boot 2.6.16.60-0.113. Sun Oct 19 06:50 (7+22:09)
	root pts/0 10.50.164.31 Sun Oct 19 06:17 - down (00:24)
	reboot system boot 2.6.16.60-0.113. Sun Oct 19 06:11 (00:30)


可以看到哪个用户在什么时间登录这台服务器的。

  

找到有嫌疑的用户之后呢，看一下这个用户的信息：

	cat /etc/passwd | grep xxx(用户名）

这样就可以看到用户名的信息了。


接下来查看一下这个用户的命令记录：

首先用root账户登录，然后`su 用户名`，root账户是可以直接su到用户的账户的。

然后执行：

	history

默认情况下，history的输出是不会有时间戳的，那我们就给他加上：

	export HISTTIMEFORMAT="%d/%m/%y %T "

然后再执行history，这样这个用户执行的所有命令和时间全都一览无遗了。
