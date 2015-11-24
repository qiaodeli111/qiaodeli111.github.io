---
layout: post
title: "Troubleshooting for Mobile APP intermittently slowness issue"
description: ""
category: "middleware"
---
{% include JB/setup %}

本文记录一次问题诊断的过程。

***问题描述***

如题，基于移动设备客户端的应用，其中一个获取人员列表的功能时快时慢。在测试过程中，100次中大概有10次移动客户端会出现timeout的错误。（timeout是在Mobile APP上设置的，Mobile APP设置的timeout值是30秒）

<!-- more -->

------

***环境描述***

生产环境和测试环境均出现了这个问题，这次排错使用测试环境。

![http://7rfgqc.com1.z0.glb.clouddn.com/2015/11/17diagram1.png](http://7rfgqc.com1.z0.glb.clouddn.com/2015/11/17diagram1.png)

上图描述了整个系统中的关键节点，以及服务器如何接收请求，生成相应并且交付到移动应用上。

客户的请求利用无线的方式发送到网络ISP，网络ISP转发到WEB服务器上。WEB服务器会把动态请求转发到APP服务器进行进一步处理。WEB服务器，APP服务器和DB服务器都在DMZ中，每个DMZ的前端都设置有防火墙。

服务器上运行的应用服务器版本如下：

- WEB： IBM HTTP Server 8.0.0.7 （基于Apache）
- APP： IBM WebSphere Application Server 8.0.0.9 (Java中间件）
- DB： Sybase

另外，APP服务器上运行多个应用，WEB服务器上也有多个虚拟主机。这个有问题的应用所在的虚拟主机里其他的应用都可以正常使用。

------

***诊断过程***

###粗略定位问题的位置

系统中节点众多，首要任务是定位问题发生在什么位置。每个请求都要经过以下节点：

*手机APP－－网络ISP－－防火墙－－WEB服务器－－防火墙－－APP服务器－－DB防火墙－－DB服务器*

排除一些绝对没问题的节点：

1. 服务器上host五个网站应用，仅仅这个应用偶尔有响应缓慢的问题，所以防火墙是没问题的。
2. 基于以上原因，服务器本身应该也是没问题的。

所以，问题发生在应用服务器上，或者网络。唯一的突破口就是日志了。

#####收集日志

WEB服务器，收集access.log和error.log。 APP服务器则收集SystemOut.log和SystemErr.log。

我们查看日志要从最后端开始查看，这个顺序相对来说最省时。


#####查看APP服务器日志

打开SystemOut.log，我们主要查看的是用户的请求有没有在APP服务器端正常处理完。对于WebSphere应用服务器（WAS）来说，我们可以从第四列看到输出的日志是由开发的应用输出的（O），还是由应用服务器本身输出的（W代表WebContainer，E代表Error）。

在我们的case中，所测试的这个应用本身有输出事务的开始和结束时间，所以比较方便我们分析。

	[11/11/15 15:36:25:782 CST] 0000004c SystemOut     O /eCOM
	[11/11/15 15:36:25:782 CST] 0000004c SystemOut     O 
	......
	[11/11/15 15:36:28:110 CST] 0000004c SystemOut     O eCOM 2015-11-11 15:36:26,110 INFO  FrameworkLog :Application Framework forwards /AppleOutputJSON.jsp

日志中可以看到，从接收到请求开始，三秒钟的时间APP服务器就可以生成相应。比较了多个时间点的请求，发现基本都是三秒钟完成相应。所以目前可以确认至少APP服务器和DB这边不存在响应慢的问题。


所以我们把问题缩小到了如下范围：

*手机APP－－网络ISP－－防火墙－－WEB服务器－－防火墙－－APP服务器*


#####查看WEB服务器日志

下面我拣出两条有代表性的WEB日志。两条日志都是客户端出现TimeOut的情况，然而两条日志有很大的区别。

我们需要看的是第四列（日志到达WEB服务器的时间点：[16/Nov/2015:21:23:26 +0800]）和第五列（WEB服务器处理这个请求所花的时间：％T，IHS和Apache默认都没有这一列，需要自己手动设置）。

这里需要做一下说明，这个％T参数所得到的时间，是指Apache从收到请求到Apache把响应数据包发送到客户机所花的时间。也就是说这个时间包含了：WEB服务器处理请求和响应的时间，APP服务器处理请求和响应的时间，数据包在WEB服务器和APP服务器之间传送的时间，响应数据包从WEB服务器传到客户机的时间。

从下面的第一条日志我们可以看到，WEB服务器仅仅花了3秒钟就得到了响应报文并且把响应报文传回给了客户端。然而即使这样客户端还是出现了timeout，也就是说有可能客户机发出的请求花了很久才把请求发送到WEB服务器上。而第二条日志，可以看到％T得到的时间是34秒。显然这个时间是过长的。

	111.161.17.67 - - [16/Nov/2015:21:23:26 +0800] 3 3921699 "POST /eCOM/AppleWebService?_command_=AppleWebService&module=HelpModule&actionType=RecordAppLog HTTP/1.1" 200 223 "-" "Mozilla/5.0 (Linux; Android 5.1.1; D6503 Build/23.4.A.1.232; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/45.0.2454.95 Mobile Safari/537.36" 
	111.161.17.67 - - [17/Nov/2015:00:37:43 +0800] 34 34972869 "POST /eCOM/AppleWebService?_timeIdND=20151117003742_76 HTTP/1.1" 200 174724 "-" "Mozilla/5.0 (Linux; Android 5.1.1; D6503 Build/23.4.A.1.232; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/45.0.2454.95 Mobile Safari/537.36" 

所以问题可能出现的原因有下面几种：

1. WEB服务器与APP服务器之间传输请求包或者响应包的时候速度过慢，造成了延时。
2. WEB服务器本身处理缓慢，造成了延时。
3. Mobile APP和WEB服务器在传输数据的时候速度过慢，造成了延时。
4. Mobile APP不能及时发出请求数据包，或者Mobile APP收到了响应数据包，但是不能快速解析响应数据包，造成了延时。


###问题排查



