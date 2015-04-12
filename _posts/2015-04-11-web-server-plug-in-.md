---
layout: post
title: "Web Server Plug-in问题诊断——文档翻译"
description: ""
category: ["middleware", "translation"]
---
{% include JB/setup %}


### 声明

本文系IBM官方WebSphere问题诊断系列文档翻译，旨在为自己增加知识和方便国人查看。本翻译遵循实用原则，原文过于拖沓啰嗦的地方就略过啦。有兴趣的可以继续阅读，没兴趣的可略过。谢谢。

本文为翻译plugin问题诊断文档：redp4045-WebSphere Application Server V6 Web Server Plug-in Problem Determination.pdf，次文档可以从这里下载：[文档下载](http://dellyqiao.qiniudn.com/2015/04/11/redp4045-WebSphere Application Server V6 Web Server Plug-in Problem Determination.pdf)

下面进入正题：

<!-- more -->

## WebSphere Application Server V6: Web Server Plug-in Problem Determination

本文讨论一下跟WebSphere Application Server V6（后文简称WAS）相关的Web服务器的plugin-in的问题诊断技术，包括由配置错误引起的问题。Web服务器plugins用来处理Web服务器跟Web container（Web container是应用服务器上的组件）之间的连接交互。跟plugin相关的问题有以下标志：

- 用户无法通过Web服务器访问到应用
- 负载均衡和故障转移（failover）无法正常工作
- 会话数据丢失
- 应用响应速度缓慢或时快时慢
- 在安装或配置plugin之后，Web服务无法启动

Plugin可以与六中不同的Web服务器协同工作，但是本文只关注IBM HTTP Server（后文简称IHS）和它的plugin。本文不会去讨论Web服务器本身的问题或者跟Web Container相关的问题。

> Tip：强烈建议您从应用服务器开始排错的过程，请阅读：Approach to Problem Determination in WebSphere Application Server V6 at http://www.redbooks.ibm.com/redpapers/pdfs/redp4073.pdf（以后将继续翻译）

### 介绍

图1展示了plugin如何在Web服务器和应用服务器上的Web Container之间进行交互。注意，Web服务器和应用服务器不需要都放到同一台物理服务器上。

![](http://dellyqiao.qiniudn.com/2015/04/11/figure1.png)

根据图示，本文讲到的问题诊断技术的关注点在Web服务器plugin和它的配置文件上。由于安装plugin的时候也需要修改Web服务器配置，创建plugin的时候需要使用WebSphere管理工具，因此本文还会讨论到这些部分。


我们需要关注下列用户可以看到的问题的标志：

- 用户无法访问到应用（页面无法找到，和页面无法显示）
- 会话数据丢失（比如每次都需要登录，购物车数据丢失）
- 响应速度慢，时好时坏

我们还需要关注下列系统管理员可以看到的问题的标志：

- Web服务器无法启动
- 在集群环境中应用无法正确地分布到集群成员服务器上

所有可以使用plugin的Web服务器都可以在运行时加载外部模块；plugin也正是其中的一个模块。比如，IHS使用两行命令加载plugin模块，引用plugin配置文件。如下面的Example 1。

Plugin配置文件是一个XML文档，它告诉Web服务器如何处理那些需要发送到WAS服务器的请求。

> Example 1 IBM HTTP Server plug-in directives
	LoadModule was_ap20_module "C:\IBM\WAS6\Plugins\bin\mod_was_ap20_http.dll"	WebSpherePluginConfig "C:\IBM\WAS6\Plugins\config\webserver1\plugin-cfg.xml"**Plugin的工作原理是这样的：它拦截所有Web服务器收到的请求，然后用URL里的Host name和端口去跟Plugin配置文件中定义的`VirtualHostGroup`和`UriGroup`里定义的URI相比较。如果找到了对应的`VirtualHostGroup`和`UriGroup`，那么plugin会继续寻找哪个`Route`会使用所找到的`VirtualHostGroup`和`UriGroup`。在找到`Route`之后，plugin会把请求发送到`Route`中定义`ServerCluster`上。**

在根据以上的工作原理，在Example 2中，URL请求`http://server/snoop`将会被转到WAS，其他的所有请求将会被传到Web服务器继续处理。

> Example 2 Excerpt from the plug-in config file
	<VirtualHostGroup Name="default_host">		<VirtualHost Name="*:80"/>	</VirtualHostGroup>	<UriGroup Name="default_host_cluster1_URIs">		<Uri AffinityCookie="JSESSIONID" AffinityURLIdentifier="jsessionid" Name="/snoop/*"/>	</UriGroup>	<Route ServerCluster="cluster1" 
	UriGroup="default_host_cluster1_URIs" 
	VirtualHostGroup="default_host"/>
	
Plugin配置文件由WebSphere administrative console管理，只要你有任何影响到plugin的更改，就需要重新生成plugin。这种更改包括：

- 修改Virtual host定义
- 更改HTTP传输类型或端口
- 创建或删除服务器
- 安装或卸载应用

> Tip: 你可以设置自动生成和传输plugin配置文件。方法请自行查阅。

#### 负载均衡

Plugin可以为多台集群成员实施负载均衡。

Figure 2 展示了一种负载均衡的例子，Plugin安装到了Web服务器所在的主机，通过round-robin或random方式把请求转到后面的两台应用服务器上。

![](http://dellyqiao.qiniudn.com/2015/04/11/figure2.png)

[点击这里](http://publib-b.boulder.ibm.com/abstracts/sg246392.html)查看更多关于可扩展性和性能方面的内容功能。

plugin-cfg.xml文件除了描述哪些URL应该被route到plugin之外，它还描述了服务器集群。Example 2展示了snoop URL相关的那些配置，这些配置告诉plugin，snoop应用程序应该由叫做`default_host_cluster1_URIs`的`UriGroup`去处理。

Example 3展示了`route`配置，这些配置标明`default_host_cluster1_URIs`里提到的所有应用将会被叫做`cluster1`的这个集群去处理。`cluster1`这个集群拥有两台服务器，第一台要通过HTTP端口9080和HTTPS端口9443去访问，第二胎要通过HTTP端口9081和HTTPS端口9444去访问。
> Example 3 Clustering excerpts from the plug-in configuration file
	￼￼￼￼￼<Route ServerCluster="cluster1" UriGroup="default_host_cluster1_URIs" VirtualHostGroup="default_host"/>	...	<ServerCluster CloneSeparatorChange="false" LoadBalance="Round Robin" Name="cluster1" 
	PostSizeLimit="-1" RemoveSpecialHeaders="true" RetryInterval="60">	<Server CloneID="10ig7jdvd" ConnectTimeout="0" ExtendedHandshake="false" 
	LoadBalanceWeight="2" MaxConnections="-1" Name="kll6571Node01_server1" ServerIOTimeout="0"
	WaitForContinue="false">         <Transport Hostname="kll6571" Port="9080" Protocol="http"/>         <Transport Hostname="kll6571" Port="9443" Protocol="https">            <Property Name="keyring" Value="C:\ibm\was6\plugins\etc\plugin-key.kdb"/>            <Property Name="stashfile" Value="C:\ibm\was6\plugins\etc\plugin-key.sth"/>         </Transport>	</Server>	<Server CloneID="10ig7jfqi" ConnectTimeout="0" ExtendedHandshake="false" 
	LoadBalanceWeight="2" MaxConnections="-1" Name="kll6571Node01_server2" ServerIOTimeout="0" 
	WaitForContinue="false">         <Transport Hostname="kll6571" Port="9081" Protocol="http"/>         <Transport Hostname="kll6571" Port="9444" Protocol="https">            <Property Name="keyring" Value="c:\ibm\was6\plugins\etc\plugin-key.kdb"/>            <Property Name="stashfile" Value="c:\ibm\was6\plugins\etc\plugin-key.sth"/>         </Transport>	</Server>

这些配置可以使snoop的请求被转到某台应用服务器上，可能是监听9080的那台也可能是另一台。

Plugin还会监控它发给应用服务器的请求，如果任何一台服务器无法连接，那么那台服务器将会被标记为宕机，同时会把它移出集群，直到它恢复正常。

负载均衡相关的配置，一部分通过WebSphere Console配置，一部分通过直接修改plugin-cfg.xml文件来配置。[点击这里](http://publib.boulder.ibm.com/infocenter/wasinfo/v6r0/index.jsp?topic=/com.ibm.websphere.nd.doc/info/ae/ae/crun_srvgrp.html)查看更多相关内容。


#### 会话亲缘性

J2EE应用支持会话的概念。会话是一种维护某个用户所有请求的方法。举例来说，我们用会话可以维护用户的购物车清单。

会话是在Web Container管理的。然而plugin具有维护会话亲缘性的功能。当用户连接到使用会话的应用时，比如购物车，Web Container将会启动会话，并且把用户的session ID回传到浏览器端，一般会放到cookie中。

集群环境中维护会话的方式很多（比如，把每个请求的会话数据写入到一个共享的数据库）。如果使用了会话亲缘性，plugin将会把某个用户所有的请求都转到创建会话的那台应用服务器。Plugin是通过在内部维护服务器和session ID的列表来实现这个功能的。使用会话亲缘性为应用程序提供了最好的性能表现，因为你不必再为每个请求读取或写入数据库，而是把会话数据都放到了应用服务器的内存里。

想要使用会话亲缘性无须堆plugin进行任何配置。只要在应用服务器中启用了会话支持，并且plugin配置文件中有`CloneID`参数就可以了：

	<Server CloneID="10ig7jdvd" ...>

默认情况下会生成`CloneID`，这是plugin识别没太应用服务器要用到的东西。

***注：实际上`CloneID`参数只会在集群环境中生成，在Standard版本的WAS中生成plugin配置文件的时候是不会产生`CloneID`参数的，因为后段的应用服务器只有一台，WAS也没必要去做这个额外的配置。但是可以通过在Web Container进行一些配置强制生成`CloneID`。***

### 解决问题

诊断问题从收集数据开始，这里有一个你需要收集的文件列表，以及介绍如何收集他们。如果你被限制不能重现问题，你就需要把所有文档一次集齐。

接下来，你需要过一下一系列问题，这些问题描述了一些问题的特征，根据特征去寻找问题的根源，每一步都能让你更接近问题的根源。

最后，我们提供了解决问题的指导文档，这些文档可能是一个网站，可能是去联系IBM，或者配置某些东溪，或者其他建议。

#### 收集数据

跟问题相关的日志和trace包括：

- Web服务器日志
- Web服务器plugin日志
- Plugin trace （可以从plugin日志里找到）
- WAS SystemOut和SystemErr日志
- WAS console的信息
- 网络协议分析（有时候是指iptrace）

##### *Web服务器日志文件*

大部分Web服务器会输出两个日志文件：包含了所有访问Web服务器的详情的access文件，和记录了错误的error文件。默认位置为：

- IHS，Apache，SunOne

	- Windows

			Access日志：<Web_server_home>\logs\access.log
		
			Error日志：<Web_server_home>\logs\error.log
		
	- Unix

			Access日志: <Web_server_home>/logs/access_log
					Error日志: <Web_server_home>/logs/error_log
		
- 微软IIS

	- Windows

			Access日志: C:\WINNT\SYSTEM32\LogFiles\W3SVC1\<date>.log 
		
			Error日志: Windows事件日志（eventvwr）
		
- Domino Web server

	Domino Web服务器日志放在数据库里，请自行查阅Domino文档了解更多。
	
##### *Web服务器plugin日志*

plugin也会输出自己的日志，日志文件会输出到plugin安装目录下的某个目录。可以从plugin配置文件中找到日志的路径，如Example 4.

> Example 4 Location of plug-in log file	<Log LogLevel="Error"	Name="c:\ibm\was6\plugins\logs\webserver1\http_plugin.log" />默认的LogLevel是`Error`，你可以设置为`Trace`来收集更详细的跟踪信息。

##### *Plugin trace*

要获取有效的跟踪信息，你需要尽可能多地得到日志。比如，你可以为IHS设置配置文件中的LogLevel为`debug`来获得更详细的日志输出：
	
	LogLevel debug

对于这个更改，你必须重启IHS服务才能生效。

同时可以在plugin里启用trace日志，修改plugin-cfg.xml文件即可：

	<Log LogLevel="Trace"	Name="c:\ibm\was6\plugins\logs\webserver1\http_plugin.log" />
	
对于这个更改，不需要重启IHS即可生效。

> plugin trace将会显著增加数据量，所以启用后尽量测试具体的问题一减少日志的行数。

##### *WAS日志*

从应用服务器上获取，日志文件的路径为：

	<WAS_install_root>/profiles/<profile>/logs/<server>/SystemOut.log 
	<WAS_install_root>/profiles/<profile>/logs/<server>/SystemErr.log

##### *网络跟踪信息*

少量情况下，你需要使用网络分析工具获取iptrace，这可以帮助检测出问题的位置。WAS不支持这种工具，你可以使用第三方工具完成这项工作。如 http://www.ethereal.com/


#### 查看问题特征

我们通过查看下面列表中提到的问题特征描述来找到应对的方案：

- 无法从Web服务器获得响应

	检查Web服务器是否已启动，可以检查进程或者访问顶级URL。例如：
	
		http://localhost/
	
	如果无法得到Web服务器的欢迎也，检查一下Web服务器的进程是否已经启动，如果没有，尝试手动启动。
	
	如果Web服务器无法启动，查看 [“Problem: Web server will not start”](#problem1)
	
- Web服务器已经启动，但是无法通过它访问应用。

	比如说用这个URL `http://Web_server/snoop`去访问snoop servlet无法正常工作。
	
	这时要先检查一下应用是否可以通过Web Container直接访问，使用这种URL去检测：`http://Application_server:WC_port/snoop`
	
		要通过Web Container直接访问应用：
		1. 找到对应的Web Container的端口：
			a. 在WAS console中，选择 Servers -> Application Servers.
			b. 点击服务器名
			c. 在“Communications”区，展开“Ports”
			d. 记住“WC_defaulthost”所对应的端口号
		2. 用浏览器使用端口号去访问对应的资源。
			例如：如果端口号是9080，那么URL就是：http://localhost:9080/snoop
			
	如果可以通过Web Container访问到应用但是通过Web服务器就不行，那么查看 [“Problem: Failure between the Web server and plug-in”](#problem2)

- 应用依赖于会话，但是会话信息很明显丢失了。

	例如，如果你每次访问应用时都要求登录，或者购物车数据丢失，那就说明有可能发生了会话丢失的现象。这种情况查看 [“Problem: Sessions are being lost”](#problem3)

- 集群环境下，应用时好时坏

	例如，2/3的请求是工作的，另外的1/3会超时。这种情况查看 [“Problem: The application works intermittently”](#problem4)
	
- 应用负载没有均匀地分布到集群中的成员上

	例如，某台集群成员CPU负载在80%，另一台负载总是很小。这种情况请查看 [“Problem: Application load is not being evenly distributed”](#problem5)
	
如果上面的列表里没有你遇到的问题，查看这里 [“The next step”](#problem6)

#### 分析问题

根据收集到的信息，你应该已经被指引到了下面的某一个问题类别里。如果没有的话，查看 [“The next step”](#problem6)

<h4 id="problem1">Problem: Web server will not start</h4>

如果你已经安装了plugin，或者生成了plugin配置文件，然后Web服务器无法启用，那么问题应该在plugin上。

##### 需要收集的数据

需要收集下列日志以确定为何无法启动Web服务器，请把日志拷贝出来以防在debug的过程中日志被覆盖。

- OS操作系统信息
- Web服务器日志
- plugin日志

在某些情况下，错误信息不会输出到任何日志中，这时你需要用命令行启动应用然后观察在控制台输出的错误信息。

##### 应该看哪些东西

查看标识出为什么服务无法启动的日志。如果看到最后都没有失败的信息，那么尝试启动服务器然后直接查看输出的信息。

Example 5展示了如果plugin-cfg.xml文件缺失，日志会是什么样：

> Example 5 Starting the HTTP Server from the command line	
	C:\IBM\HTTP\bin>apache	ws_common: websphereUpdateConfig: Failed to stat plugin config file for
	C:\IBM\WAS6\Plugins\config\webserver1\plugin-cfg.xml

在Unix环境中，当你启动Web服务器的时候，一般可以在命令行中看到相关的信息。比如plugin模块被损坏了，输出如下：

> Example 6 Messages indicating a corrupt plug-in module on UNIX	Syntax error on line 844 of /opt/IBMIHS/conf/httpd.conf:	Cannot load /opt/IBM/WAS6/Plugins/bin/mod_was_ap20_http.so into server:	/opt/IBM/WAS6/Plugins/bin/mod_was_ap20_http.so: undefined symbol: ap_palloc

在Windows环境中，用Windows services面板启动Web服务器的时候很少会看到有用的信息，你需要查看日志文件。累死Example 6的信息将会出现在http_plugin.log文件中。

如果是plugin是Web服务器无法启动，那么错误信息一般会很精确很具体。等你解决了那些问题的时候就可以启动Web服务器了。

如果错误信息指向某些文件，那么请确认文件确实存在并且没被损坏。确认过程中请使用启动Web服务器的那个账户，以确保不存在任何权限问题。

如果错误信息指出跟plugin-cfg.xml相关，可以尝试重新生成这个文件。

Plugin只在特定版本的Web服务器上工作，如果你的plugin版本过劳，或者Web服务器或者GSKit版本过来，那么你需要升级一下对应的组件。下面的链接可以查看软件版本的适配信息：[http://publib.boulder.ibm.com/infocenter/wasinfo/v6r0/index.jsp?topic=/com.ibm.websphere.base.doc/info/welcome_base.html](http://publib.boulder.ibm.com/infocenter/wasinfo/v6r0/index.jsp?topic=/com.ibm.websphere.base.doc/info/welcome_base.html)

> Tip: 运行GSKit版本命令以获取版本信息，此命令在<gskit_install>/bin目录下。
> 示例：C:\Program Files\IBM\gsk7\bin\gsk7ver

如果上面的信息都跟你的情况不服，那么问题就不是有plugin引起的。这种情况下，你需要回顾一下问题的特征，确定是否应该属于别的情况。


<h4 id="problem2">Problem: Failure between the Web server and plug-in</h4>

