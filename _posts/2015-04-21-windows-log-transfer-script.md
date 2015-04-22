---
layout: post
title: "Windows Log transfer script"
description: ""
category: "server"
---
{% include JB/setup %}

4月21日更新：

本文分享一个Windows服务器日志传输的脚本。作为示例，这个脚本可以被修改为在Windows服务器之间传输任何文件的一个应用。下面具体描述一下我的环境，需求和脚本的实现方法。


<!-- more -->

### 需求

我有三台Windows服务器在运行Java应用，每台服务器上都有多个JVM，每个JVM的日志的路径也不同。而四台服务器是作为水平集群的，每台服务器上的日志路径和命名方式也都是一样的。

经常性的需要把某个特定的JVM在四台机器上的日志都取出来放到另外的一台服务器上，所以现在写个脚本实现这个任务。

### 实现过程

首先写一个Map硬盘盘符的脚本 map_drive.bat.


	net use H: /delete
	net use I: /delete
	net use J: /delete

	net use H: \\192.168.1.65\logs$ password /user:username
	net use I: \\192.168.1.66\logs$ password /user:username
	net use J: \\192.168.1.67\logs$ password /user:username


然后写一下获取日志的脚本 getlog.bat.

日志的格式也需要总结一下：

- 每台服务器上的日志路径是这样的：

	> \<logdrive\>\logs\WebSphere\sample1-server / sample2-server / sample3-server
	> 
	> \<logdrive\>\log\Websphere\sample4-server

- Java日志的命名格式是这样：

	> SystemOut.log SystemErr.log
	> 
	> SystemOut\_15.03.30\_10.26.32.log SystemErr\_15.03.30\_10.26.32.log 
	> 
	> //15.03.30代表15年3月30日，10.26.32代表10点26分32秒生成的日志
	> 
	> SystemOut\_15.03.30\_10.26.32.log.gz SystemErr\_15.03.30\_10.26.32.log.gz

- 日志轮转机制的设置是，如果日志大于50MB，就会生成一个新的日志文件。所以，当需要获得某一天的日志文件的时候，需要获取文件名为`SystemOut_YY.MM.DD*`和`SystemErr_YY.MM.DD*`的所有文件，如果需要获取今天的日志，还需要额外拿一下文件名为`SystemOut.log SystemErr.log`的这两个文件。


下面是`getlogs.bat`的具体内容：

	echo on
	call map_drive.bat

	set /p appname="Input APP Server Name (sample1/sample2/sample3/sample4):"
	##获取应用服务器名

	set logpath=%appname%-server
	##拼接日志路径字符串
	
	set destpath=logs\WebSphere\
	##设置存有日志的远程主机的日志路径

	if %appname%==sample4 (
	set destpath=log\websphere\
	set logpath=iPOSAppSrv
	)
	##为sample4设置例外
	
	set localpath=D:\SFTP_SITE\AIA_Thailand\App_log\
	##设置当前主机，日志将要存放的目标路径

	forfiles /p %localpath%%logpath% /s /m *.* /c "cmd /c del @path"
	##清理临时目录


	set /p time="Input date(eg:YY.MM.DD / today): "
	##获取想要得到的日志的日期

	if %time%==today (
	##如果需要的是今天的日志，那就得到今天的日期并且拼接成SystemOut_YY.MM.DD*格式的字符串，然后直接把SystemOut.log和SystemErr.log传输过来
	
	set sysout=SystemOut_%date:~12,2%.%date:~4,2%.%date:~7,2%*
	set syserr=SystemErr_%date:~12,2%.%date:~4,2%.%date:~7,2%*
	
	copy H:\%destpath%%logpath%\SystemErr.log %localpath%%logpath%\app10
	copy I:\%destpath%%logpath%\SystemErr.log %localpath%%logpath%\app11
	copy J:\%destpath%%logpath%\SystemErr.log %localpath%%logpath%\app12

	copy H:\%destpath%%logpath%\SystemOut.log %localpath%%logpath%\app10
	copy I:\%destpath%%logpath%\SystemOut.log %localpath%%logpath%\app11
	copy J:\%destpath%%logpath%\SystemOut.log %localpath%%logpath%\app12
	
	set time=%date:~12,2%.%date:~4,2%.%date:~7,2%
	##把SystemOut.log和SystemErr.log传输完之后，把time变量设成日期，后面做压缩包的时候会用到。

	) else (
	##如果需要的不是今天的日志，则直接根据用户输入拼接日志名字符串
	set sysout=SystemOut_%time%*
	set syserr=SystemErr_%time%*
	)

	copy H:\%destpath%%logpath%\%syserr% %localpath%%logpath%\app10
	copy I:\%destpath%%logpath%\%syserr% %localpath%%logpath%\app11
	copy J:\%destpath%%logpath%\%syserr% %localpath%%logpath%\app12

	copy H:\%destpath%%logpath%\%sysout% %localpath%%logpath%\app10
	copy I:\%destpath%%logpath%\%sysout% %localpath%%logpath%\app11
	copy J:\%destpath%%logpath%\%sysout% %localpath%%logpath%\app12
	##传输日志


	7z a %localpath%%logpath%_%time%.zip %localpath%%logpath%\
	##压缩整个文件夹

	forfiles /p %localpath%%logpath% /s /m *.* /c "cmd /c del @path"
	##再次清理临时文件夹

	explorer.exe %localpath%
	##打开放置压缩包的目录方便操作

	exit


执行getlog.bat就可以获取日志了。期间需要输入想要获取的日志的JVM名，然后输入日期，即可获得一个打包好的日志文件包了。