---
layout: post
title: "IBM WAS 6.1 Web container problem determination——文档翻译（更新中）"
description: ""
category: ["middleware", "translation"]
---
{% include JB/setup %}

### 声明

本文为IBM官方WebSphere问题诊断系列文档翻译，旨在为自己增加知识和方便国人查看。本翻译遵循实用原则，原文过于拖沓啰嗦的地方就略过。有兴趣的可以继续阅读，没兴趣的请路过。欢迎留言吐槽，谢谢。

本文为翻译plugin问题诊断文档：redp4045-WebSphere Application Server V6 Web Server Plug-in Problem Determination.pdf，次文档可以从这里下载：[文档下载](http://dellyqiao.qiniudn.com/2015/04/14/redp4309-WebSphere Application Server V6.1Web container problem determination.pdf)

下面进入正题：

<!-- more -->

## WebSphere Application Server V6.1: Web container problem determination

Web组件的运行时环境叫做Web容器（Web Container）。当用户无法正常访问系统时，可能就是Web Container的问题。

对于Web container来说，可能出现的问题有：

- 用户无法访问Web组件
- 运行Web组件的时候反应不正常
- 启动Web组件时报错
- JSP进行编译的时候有错
- 运行Web组件时抛出错误或异常
- 收到以SRVE（Web container）、JSPG（JSP）、JSFG（JSF页面）开头的信息

本文教你如何在分布式系统和IBM i5/OS系统上诊断IBM WAS V6.1的Web container的问题。

### Web容器介绍

Web容器是运行Web应用的运行时环境，它处理servlet，JSP文件，和其他种类的服务器端组件。每个应用程序服务器运行时都有有一个单独的逻辑Web容器，你可以修改它，但是不能创建或删除。

### Web容器概述

Figure 1图解了Web容器，以及它在应用服务器中的位置。

![](http://dellyqiao.qiniudn.com/2015/04/14/1.png)

Web容器提供：

- Web容器传输链（transport chains）

	Web容器inbound传输链用来处理请求。它由下列组件组成：用来提供容器和网络之间的链接的TCP inbound channel，用来为HTTP 1.0和1.1提供服务的HTTP inbound channel，还有用来把servlet和JSP请求发送到Web容器的Web container channel。

- Servlet处理进程

	当Web容器处理servlet请求时，Web容器会创建请求对象和响应对象，然后调用servlet的service方法。当时机合适，Web容器也会调用servlet的destroy方法，之后JVM会执行垃圾回收（GC）。

- HTML和其他静态内容的处理进程

	当Web容器接受到HTML或其他静态内容请求时，Web容器inbound传输链会提供服务。然而，在生产环境下，使用外部Web服务器并且使用plugin作为接口在大多数情况下是更为合适的选择。
	
- 会话管理

	Web容器支持API中定义的javax.servlet.http.HttpSession接口规范。

### Web应用程序

Servlet和JSP文件就是所说的Web组件。静态内容文件（比如HTML页面，图片，XML文件）在进行应用封装创建Web模块的时候会跟Web组件捆绑在一起。Web模块是一种遵循指定的格式规范去组织和打包的，可以被单独拿出来部署和使用的一个单元，它会被打包成Web archive（WAR）文件。J2EE Web模块符合Java servlet规范。

Web模块最顶层的目录是应用程序的`document root`，也就是放置JSP页面和静态Web资源的位置。document root有个名为`/WEB-INF/`的子目录，其中放置Web应用部署描述符（web.xml）和Web模块需要用到的服务器端的类文件。如Figure 2：

![](http://dellyqiao.qiniudn.com/2015/04/14/2.png)

### 确认Web容器的问题类型

Web容器出问题的时候会给用户返回很多种类的标识符，比如HTTP 404错误，500错误，或者页面上显示不正确的信息，例如由HTTP错误引发的一个错误页面，或者页面上显示的内容不正确，或者页面显示不完整。

当发生HTTP错误的时候，Web浏览器会显示一个包含错误码的错误页面。如果在Web模块中自定义了错误页面，那你需要检查Web服务器日志以确定真实的HTTP错误码是什么，因为那些错误码很可能并不会显示在你自定的页面上。

#### 收集分析Web服务器日志

从Web服务器上手机access日志，然后搜索日志中的404或500错误。下面是一些IHS（IBM HTTP Server）的例子：

- access.log日志：

		127.0.0.1 - - [28/Mar/2007:19:52:31 -0400] "GET url HTTP/1.1" 404 304 
		127.0.0.1 - - [28/Mar/2007:20:03:48 -0400] "GET url HTTP/1.1"500 5348
	
	重点注意一下跟出问题的URL相关的错误码。

- 如果正常情况下从Web服务器应该可以访问某个URL（这个请求需要到达应用服务器进行处理），然后出现了类似这样的信息：从xxx地方无法找到指定的文件。这时需要查看error日志，找一下文件无法访问的日志：

		- [Wed Mar 28 19:52:31 2007] [error] [client 127.0.0.1] File does not exist: file_location

### HTTP 404错误

HTTP 404错误有多重成因，下面是一些例子：

- 外部因素，比如Web服务器有问题
- 配置问题，比如plugin或者virtual host配置不正确
- 运行时环境的问题，比如应用或应用服务器没有启动
- 用户或应用问题，比如输错了URL

#### 验证系统的完整性

如果用户得到404错误并且你比较了解出问题的应用程序，那么首先应该检查一下系统的完整性，需要检查的组件包括：

- Web服务器
- 应用服务器
- 应用程序

如果这些组件都工作正常，或者你不是很清楚哪台应用服务或哪个应用受到了影响，请继续往下看。

##### ***确认Web服务器是否正常响应***

如何检查？取决于Web服务器的种类和它的配置。

如果是IHS，简捷的方法是从浏览器访问对应的URL：http://server_name。如果可以看到欢迎页面，说明Web服务器可以工作。如果无法显示，出现了“page cannot be displayed”这类的信息，说明问题可能处在Web服务器。

##### ***解决Web服务器问题***

参阅另一篇文章：[http://blog.dellyqiao.com/middleware/translation/2015/04/11/web-server-plug-in-/](http://blog.dellyqiao.com/middleware/translation/2015/04/11/web-server-plug-in-/)

##### ***确认应用服务器是否启动***

1. 确认应用程序是否部署在了服务器或集群上
2. 确认应用服务器的状态
3. 如果没启动，尝试启动一下

如果你不知道这些步骤怎么做，查看这里：[]()

##### ***确认应用程序是否启动***

从管理控制台查看，Applications → Enterprise Applications。

如果应用状态是启动，说明应用在运行，但是它无法检测应用在启动过程中是否有任何错误。

#### 收集诊断相关的数据

如果你无法看到错误的类型或者无法得知出问题的URL信息，你需要手机以下数据：

- 用户浏览器显示的错误以及失败的URL
- 应用服务器的日志（如果应用部署在集群上，需要手机每台集群成员的日志）
- Web服务器access和error日志
- TRCTCPAPP trace（i5/OS）

	i5/OS系统上的HTTP服务器是集成到系统里的，还有一种额外的跟踪日志可以提供有用的信息。查看这里：[]()

#### 分析数据

从诊断数据里能得到的信息主要是这两种：

- 出问题的URL
- 问题出在哪儿（Web服务器，plugin，应用服务器）

可以以下面的顺序做问题诊断：

1. Web浏览器显示的错误信息

	这些信息可以帮助你得知问题的类型和出问题的URL。这些信息有可能已经足够解决你的问题。

2. SystemOut日志

	如果是Web容器出问题，SystemOut日志是最能提供有用信息的文件。日志中的信息也有可能会显示到浏览器中，取决于应用程序如何处理错误。如果这个JVM日志中没有任何错误信息，多数可能是因为请求没有到达应用服务器。
	
3. TRCTCPAPP trace（只针对i5/OS系统）

	这个trace可以帮你i安测问题是否出在plugin上。
	
4. Web服务器日志

	如果没有发现其他的错误迹象，查看一下Web服务器日志，卡伊找到错误的类型（404、500）和错问题的URL。


#### 分析浏览器输出的错误信息

一般来说，404错误会伴随有一些描述性的问题说明信息一起出现，如果用户报告的问题属于下面列表中的任何一条，我们就可以直接定位问题原因：

- The page cannot be displayed
- JSP error or JSF error
- Failed to find resource
- File not found
- WebGroup/virtual host not defined

