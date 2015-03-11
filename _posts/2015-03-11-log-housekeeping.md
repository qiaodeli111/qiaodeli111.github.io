---
layout: post
title: "Log housekeeping日常维护之日志管理"
description: ""
category: ["Server", "Middleware"]
---
{% include JB/setup %}

服务器的日常维护总需要维护日志，常常用到的命令无非就是两种：

#### 1）把N天以前的日志全部删掉

Windows系统用forfiles：

	forfiles /s /m *.log /d -60 /p “D:/logs/“ /c “cmd /c del @file"

参数解释：

  * /s 搜索子文件夹
  * /m mask,文件的标志符号
  * /d date，-60就是指60天以前
  * /p path，要做搜索的一个路径
  * /c 后面要跟上需要执行的操作 

  
<!-- more -->

Linux系统用find命令：

	find /var/log/ -name .log -mtime +60 -exec rm -rf {} \;

具体的用法可以man find来获取，find工具太博大精深了，拣自己需要用到的看一下就好了。


#### 2）把当前的log文件压缩一下

压缩需要配合应用服务器的log输出的格式，比如Websphere可以设置每天rotate一次日志，定时把第一天的日志变成 SystemOut-2014-11-27.log类似这样的格式。这样的话我们借助脚本只把已经归档出来的日志压缩一下就行，然后设置好定时执行就好了。

- Windows系统:

	需要使用工具了，下载个gzip.exe，zip.exe之类的，然后写一个脚本，类似于这样：

		G:
		cd G:\logs\WebSphere\server1

		For /F %%F in ('dir /b G:\logs\WebSphere\server1\\*.log') DO (
		if NOT %%F == SystemErr.log if not %%F == SystemOut.log IF NOT EXIST %%F.gz G:\gzip.exe -q -9 %%F
		)

	这样，Websphere正在使用的SystemOut.log和SystemErr.log文件就不会受到影响，其他的日志一概压缩归档。如果需要定时执行的话设置个计划任务就好。

- Linux系统：

	选择比较多，一般Linux系统都自带gzip，tar这类的归档压缩工具，比如可以用这个命令：`gzip SystemOut-*.log`， 直接把归档出来的日志都压缩。如果需要定时执行的话设置一下crontab就好了。

	Crontab的设置如下：

	1. `crontab -e` 进入编辑模式。crontab的编辑模式使用的时Vi编辑器的操作方式，在编辑之前先弄懂vi怎么操作，否则麻烦很大。
	2. 添加这一行：

			10 0 * * * gzip /var/log/SystemOut-*.log >/dev/null 2>&1

		代表每天0点10分自动执行这个gzip /var/log/SystemOut-*.log命令，很简单吧。


还有另外一种比较特别的情况：

**3）Apache 服务器的access.log的清理：**

如果Apache服务器恰好没有设置自动归档，那随着网站访问量增长，access日志会非常大，如果想清理日志又不希望宕机的话，可以采用下面的办法：


仅针对Linux，切记。 Windows怎么做我不知道。。。。。。


	mv access_log access_log.old

	kill -1 `cat httpd.pid`

这个httpd.pid就是进程号，这种方法可以不重启整个进程而仅仅启动需要启动的线程，就可以创建出来新的access_log并且不宕机了。

  


  

