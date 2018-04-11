---
layout: post
title: "DoubleTake replication via command line"
description: ""
category: "Server"
---
{% include JB/setup %}

废话不多说，DoubleTake文件传输的命令行配置方法如下：



	d:\DoubleTake>DTCL.exe -i

	Command:  logon 10.52.21.51 csc_mcs "********"
	  User access level set to DT_FULL_ACCESS
	Command:  logon localhost csc_mcs "********"
	  User access level set to DT_FULL_ACCESS
	//上面两条命令时登陆源服务器和目标服务器，确保用户访问权限是“DT_FULL_ACCESS"，如果是别的字样，说明权限有问题，需要把账户添加到"Double-Take Admin"这个组。  

	Command:  source localhost
	Command:  target 10.52.21.51
	//指定源服务器和目标服务器。数据会从源到目标，如果设置错误，后果很危险。

	Command:  repset create newrep
	//创建一个新的replication set，也就是要同步的数据集。
	Command:  repset rule add "d:\" include to newrep
	Command:  repset rule add "d:\DoubleTake" exclude to newrep
	//为这个数据集添加规则，比如我就是需要同步D盘下所有文件，但是不要同步D:\DoubleTake这个目录。
	Command:  repset save
	Command:  repset list
	  - List of rep sets -
	  newrep                                              disabled

	Command:  connect newrep to 10.52.21.51 map exact
	//建立连接，连接之后数据就会开始传输了。
	//注意上面的map exact代表完全按照路径去映射，比如源路径是d:\，那么文件也会传到目标服务器的d:\。
	//当需要映射到不同的目标时，使用这个命令：
	//connect datafile to 10.52.21.57 map d:\ to e:\

	Command:  status con 1
	  - Status for connection 1 -
	  Time established:                 08/17/2015 13:59:07.473
	  Replication set:                  newrep
	  Target machine:                   10.52.21.51
	  Target state:                     Mirror Required
	  Mirror bytes transferred:         0
	  Mirror bytes transmitted:         0
	  Mirror bytes in queue:            0
	  Replication bytes transferred:    2040
	  Replication bytes transmitted:    2040
	  Replication bytes in queue:       0
	//这个命令可以看到当前这个链接的状态。一般来说建立的第一个链接编号就是1.

	Command:



附上一份DoubleTake命令指南：
[DoubleTake Command Guide](http://7rfgqc.com1.z0.glb.clouddn.com/2015/08/17/double-take-scripting-guide.pdf)