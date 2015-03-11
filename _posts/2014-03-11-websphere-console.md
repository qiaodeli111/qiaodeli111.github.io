---
layout: post
title: "如何重新部署Websphere Console控制台"
description: ""
category: "Middleware"
---
{% include JB/setup %}


对于StandAlone服务器，跳转到${profile_home}/bin。

对于ND集群，控制台是部署在DMGR上的，要跳转到${WAS_HOME}/bin。

  
<!-- more -->

然后执行这个命令删除：

	wsadmin.bat -lang jython -f [deployConsole.py](http://deployConsole.py) remove  


  


执行这个命令重新安装：

	wsadmin.bat -lang jython -f [deployConsole.py](http://deployConsole.py) install  


  

