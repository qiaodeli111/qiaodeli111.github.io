---
layout: post
title: "Windows Log transfer script"
description: ""
category: "server"
---
{% include JB/setup %}

本文分享一个Windows服务器日志传输的脚本。作为示例，这个脚本可以被修改为在Windows服务器之间传输任何文件的一个应用。下面具体描述一下我的环境，需求和脚本的实现方法。

<!-- more -->

### 需求

我有四台Windows服务器在运行Java应用，每台服务器上都有多个JVM，每个JVM的日志的路径也不同。而四台服务器是作为水平集群的，每台服务器上的日志路径和命名方式也都是一样的。

经常性的需要把某个特定的JVM在四台机器上的日志都取出来放到另外的一台服务器上，所以现在写个脚本实现这个任务。

### 实现过程

首先写一个Map硬盘盘符的脚本 map_drive.bat.


	net use H: /delete
	net use I: /delete
	net use J: /delete
	net use K: /delete

	net use H: \\192.168.1.65\logs$ passwordrd /user:username

	net use I: \\192.168.1.66\logs$ password /user:username

	net use J: \\192.168.1.67\logs$ password /user:username

	net use K: \\192.168.1.68\logs$ password /user:username


然后写一下获取日志的脚本 getlog.bat.

每台服务器上的日志路径是这样的：

> www3-jvm-sample1 / sample2 / sample3
> 
> www-jvm-sample4 / sample5

Java日志的命名格式是这样：

> SystemOut.log SystemErr.log
> 
> SystemOut_15.03.30_.log SystemErr_15.03.30_.log 
> 
> SystemOut_15.03.30_.log.gz SystemErr_15.03.30_.log.gz


	echo off
	
	call map_drive.bat
	
	##获取用户输入，然后拼接日志路径的字符串到%appname%变量

	set /p appname="Input APP Server Name (sample1/sample2/sample3/sample4/sample5):"

	set logpath=www3-jvm-%appname%
	if %appname%==sample4 set logpath=www-jvm-%appname%
	if %appname%==sample5 set logpath=www-jvm-%appname%

	##获取用户输入，以确定需要传输哪天的日志
	
	set /p time="Input date(eg:YY.MM.DD / today): "

	if %time%==today (
	set sysout=SystemOut.log
	set syserr=SystemErr.log
	) else (
	set sysout=SystemOut_%time%_*
	set syserr=SystemErr_%time%_*
	)

	##清空临时目录D:\logs\www3-jvm-sampleX\，然后传输日志。临时目录下有四个子文件夹(app80/81/82/83)存放四台服务器上对应的日志。

	forfiles /p D:\logs\%logpath% /s /m *.* /c "cmd /c del @path"

	copy H:\App_Logs\%logpath%\%syserr% D:\logs\%logpath%\app80
	copy I:\App_Logs\%logpath%\%syserr% D:\logs\%logpath%\app81
	copy J:\App_Logs\%logpath%\%syserr% D:\logs\%logpath%\app82
	copy K:\App_Logs\%logpath%\%syserr% D:\logs\%logpath%\app83

	copy H:\App_Logs\%logpath%\%sysout% D:\logs\%logpath%\app80
	copy I:\App_Logs\%logpath%\%sysout% D:\logs\%logpath%\app81
	copy J:\App_Logs\%logpath%\%sysout% D:\logs\%logpath%\app82
	copy K:\App_Logs\%logpath%\%sysout% D:\logs\%logpath%\app83

	##把日志文件压缩成zip格式，文件名定义为: JVM名_日期.zip
	##7z的命令行工具要预先拷贝到脚本所在的目录下，7z可以从任何一个安装好的Windows版7z中获取，需要拷贝的文件为：7z.exe和7z.dll
	
	7z a D:\logs\%logpath%_%time%.zip D:\logs\%logpath%\

	##再次清空临时目录，最后从资源管理器窗口中打开日志目录
	
	forfiles /p D:\logs\%logpath% /s /m *.* /c "cmd /c del @path"

	explorer.exe D:\logs\

	exit


执行getlog.bat就可以获取日志了。期间需要输入想要获取的日志的JVM名，然后输入日期，即可获得一个打包好的日志文件包了。