---
layout: post
title: "IBM Web Server Plug-in问题诊断——文档翻译"
description: ""
category: ["Middleware", "Translation"]
---
{% include JB/setup %}


### 声明

本文为IBM官方WebSphere问题诊断系列文档翻译，旨在为自己增加知识和方便国人查看。本翻译遵循实用原则，原文过于拖沓啰嗦的地方就略过。有兴趣的可以继续阅读，没兴趣的请路过。欢迎留言吐槽，谢谢。

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

##### 如何检查

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

如果上面的信息都跟你的情况不符，那么问题就不是有plugin引起的。这种情况下，你需要回顾一下问题的特征，确定是否应该属于别的情况。


<h4 id="problem2">Problem: Failure between the Web server and plug-in</h4>

如果你猜想plugin有一些问题导致了Web服务器不能把HTTP请求发送到WAS服务器，那么这个部分会帮助你确定原因。

最明显的特征是可以直接访问Web Container，但是不能通过Web服务器去访问。

Error日志同时也会出现一个无法找到文件的记录，如下：

	[Tue Jun 28 15:54:43 2005] [error] [client 127.0.0.1] File does not exist:	C:/IBM/HTTP/htdocs/en_US/snoop

##### 需要收集的数据

收集这些日志文件可以帮助你查明问题的根源：

- Web服务器日志
- Plugin日志
- Plugin trace （debug模式的plugin日志）
- 网络trace （辅助plugin trace分析）
- plugin配置文件

##### 如何检查

首先确定Web服务器确实引用了plugin-cfg.xml文件，检查httpd.conf配置：

	WebSpherePluginConfig /opt/WAS6/Plugins/config/web1/plugin-cfg.xml打开这个xml文件，检查无法正常工作的应用程序的URI是否包含在其中，像Example 2那样。

***验证你是否使用了正确的配置文件***

如果应用程序的URI配置丢失，那么确认一下Web服务器上的这个配置文件是否是你生成的那个。当你生成plugin配置文件时，不论是用WAS Console还是`GenPluginCfg.sh/bat`命令，输出的信息都会告诉你生成的文件的具体位置。Figure 3显示plugin文件的位置：

![](http://dellyqiao.qiniudn.com/2015/04/11/figure3.png)

当以下条件满足时，plugin配置文件还可以自动传输到本地或远程服务器的指定位置：

- 远程Web服务器是IHS
- Plugin配置服务已经在运行
- 以下条件满足任意一条：

	- WebSphere的node agent运行在Web服务器所在的主机上
	- IHS运行在Web服务器所在的主机上，并且管理密码已经设定好了

- 自动传输插件的配置已经启用。如Figure 4：

![](http://dellyqiao.qiniudn.com/2015/04/11/figure4.png)

更多自动传输插件的信息请查阅这里： [http://publib.boulder.ibm.com/infocenter/wasinfo/v6r0/index.jsp?topic=/com.ibm.websphere.base.doc/info/aes/ae/uwsv_plugin_props.html](http://publib.boulder.ibm.com/infocenter/wasinfo/v6r0/index.jsp?topic=/com.ibm.websphere.base.doc/info/aes/ae/uwsv_plugin_props.html)

***验证是否正确地生成了plugin配置文件***

如果尝试生成plugin配置文件，但是应用的context root没有出现在其中，那么可能配置文件没有正确地生成。配置文件中的内容取决于你生成配置文件的方法。

如果你使用`GenPluginCfg.sh/bat`命令，没有使用任何参数，那么拓扑结构中所有的应用和应用服务器都会生成到配置文件中。然而，你可以通过参数缩小要包含进来的拓扑结构的部分。例如，如果使用命令行并且指定Web服务器名去生成plugin，或者使用WAS Console生成plugin，那么只有映射到那个Web服务器上的应用程序才会被包含到plugin文件中。

问题的原因可能只是因为你没有把应用程序映射到对应的Web服务器。这是用WAS Console做一下mapping就好。

***检查此错误：Virtual Host and WebGroup not found error***

如果配置文件没问题，但是“Virtual Host and WebGroup not found error”这样的错误信息返回到了浏览器中，那么这个错误是跟WAS相关的，查看此链接：[http://www.redbooks.ibm.com/redpapers/pdfs/redp4058.pdf](http://www.redbooks.ibm.com/redpapers/pdfs/redp4058.pdf)

***进一步分析：Trace和请求***

到现在为止，你确定一切配置都正常，看起来Web服务器可以接受请求，plugin可以把请求发到WAS，但是不能从应用方面得到响应。

下一个步骤，启用plugin trace，再次发送一个HTTP请求，然后检查每个日志，跟踪此HTTP请求以确定到底它是在哪个环节失败的。Figure 5展示了HTTP请求索要经过的路径：

![](http://dellyqiao.qiniudn.com/2015/04/11/figure5.png)

根据图示的路径追踪指定的请求可以告诉你问题出在哪儿。Example 9展示了一个成功的请求，这些日志记录展示了请求在各个组件之间成功地过渡。

写入到http_plugin.log文件里的第一行日志说明plugin绑定了转换URL的工具，如果一切正常，可以看到它将会绑定你的URL。Example 7摘要了几条snoop应用相关的trace日志：

> Example 7 Plug-in trace initial entries	
	[Thu Jun 23 14:35:46 2005] 0000097c 00000ae4 - TRACE: ws_uri: uriCreate: Creating uri	[Thu Jun 23 14:35:46 2005] 0000097c 00000ae4 - TRACE: ws_uri: uriSetName: Setting the name /snoop/* with score 7	[Thu Jun 23 14:35:46 2005] 0000097c 00000ae4 - TRACE: ws_uri: uriSetAffinityURL: Setting the affinity cookie jsessionid	[Thu Jun 23 14:35:46 2005] 0000097c 00000ae4 - TRACE: ws_uri: uriSetAffinityCookie: Setting the affinity cookie JSESSIONID	[Thu Jun 23 14:35:46 2005] 0000097c 00000ae4 - TRACE: ws_uri_group: uriGroupAddUri: Adding uri /snoop/* to front of list
	
在plugin完成转换工作之后，它会把版本信息写入到日志里，如下Example 8：

> Example 8 Plug-in initialization messages	
	--------------------System Information-----------------------	[Thu Jun 23 14:35:46 2005] 0000097c 00000ae4 - PLUGIN: Bld version: 6.0.0	[Thu Jun 23 14:35:46 2005] 0000097c 00000ae4 - PLUGIN: Bld date: Oct 31 2004, 11:15:26	[Thu Jun 23 14:35:46 2005] 0000097c 00000ae4 - PLUGIN: Webserver: IBM_HTTP_Server/6.0 Apache/2.0.47 (Win32)	[Thu Jun 23 14:35:46 2005] 0000097c 00000ae4 - PLUGIN: Hostname = KLL6571	[Thu Jun 23 14:35:46 2005] 0000097c 00000ae4 - PLUGIN: OS version 5.0, build2195, 'Service Pack 4'	[Thu Jun 23 14:35:46 2005] 0000097c 00000ae4 - PLUGIN:	--------------------------------------------------------------Example 9所示的日志说明plugin已经成功地转换了URL，然后设置传输方式，把请求转到了集群中的一台服务器去处理。之后，一个带有200返回码的响应成功地返回到了plugin。时间标说明应用服务器花了7秒钟去处理这个请求。

> Example 9 Web server plug-in request trace entries for snoop
	[Thu Jun 23 14:35:56 2005] 000007bc 000007d4 - TRACE: ws_common: websphereShouldHandleRequest: trying to match a route for: vhost='localhost'; uri='/snoop'	...	[Thu Jun 23 14:35:56 2005] 000007bc 000007d4 - TRACE: ws_common: websphereUriMatch: Found a match '/snoop' to '/snoop' in UriGroup: default_host_cluster1_URIs with score 6	...	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE: ws_common: websphereExecute: Executing the transaction with the app server	...	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE: lib_htrequest: htrequestWrite: Writing the request:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼18 WebSphere Application Server V6: Web Server Plug-in Problem Determination	GET /snoop HTTP/1.1	User-Agent: Wget/1.9	Host: localhost	Accept: */*
	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE:	WS-ESI="ESI/1.0+"	Connection: Keep-Alive	$WSIS: false	$WSSC: http	$WSPR: HTTP/1.0	$WSRA: 127.0.0.1	$WSRH: 127.0.0.1	$WSSN: localhost	$WSSP: 80	Surrogate-Capability:	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE: lib_htrequest: htrequestWrite: Writing the request content	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE: ws_common: websphereExecute: Wrote the request; reading the response	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE: lib_htresponse: htresponseRead: Reading the response: 5397bc	...	[Thu Jun 23 14:35:57 2005] 000007bc 000007d4 - TRACE: lib_htresponse: htresponseRead: Reading the response: 5397bc	[Thu Jun 23 14:36:04 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:36:04 2005] 000007bc 000007d4 - TRACE:	text/html;charset=ISO-8859-1	[Thu Jun 23 14:36:04 2005] 000007bc 000007d4 - TRACE:	en-US	HTTP/1.1 200 OK	Content-Type:	Content-Language:	Content-Length: 16166	[Thu Jun 23 14:36:04 2005] 000007bc 000007d4 - TRACE:	[Thu Jun 23 14:36:04 2005] 000007bc 000007d4 - TRACE: lib_htresponse:	htresponseSetContentLength: Setting the content length |16166|
	

在这之后，很快地Web服务器把得到的响应返回给浏览器，同时更新access日志，显示整个事务成功完成。如Example 10：

> Example 10 Web server access log entry for snoop	127.0.0.1 - - [23/Jun/2005:14:35:56 -0400] "GET /snoop HTTP/1.0" 200 16166
	
Web服务器在整个处理最后的过程才会写入Access日志。

根据上面的介绍，你可以根据提到的日志文件去确定问题出在哪个环节。

下面的Example 11展示了plugin标记应用服务器为宕机状态的日志。这个日志即使不是trace级别也会出现，这个信息包含操作系统返回的错误：`err=10061`，这条信息表示connection refused。想要知道这些错误信息的含义，你需要检查一下操作系统的文档。

> Example 11 Plug-in messages when a server goes down	
	[Thu Jul 07 13:53:20 2005] 00000d6c 000010b8 - ERROR: ws_common: websphereGetStream: Failed to connect to app server on host 'kll6571', OS err=10061	[Thu Jul 07 13:53:20 2005] 00000d6c 000010b8 - ERROR: ws_common: websphereExecute: Failed to create the stream	[Thu Jul 07 13:53:20 2005] 00000d6c 000010b8 - ERROR: ws_server: serverSetFailoverStatus: Marking kll6571Node01_server01 down 
	[Thu Jul 07 13:53:20 2005] 00000d6c 000010b8 - ERROR: ws_common: websphereHandleRequest: Failed to execute the transaction to 'kll6571Node01_server01'on host 'kll6571'; will try another one

如果plugin trace显示它绑定了请求并且把请求发送到了应用服务器，但是没有显示收到的回复，那么问题可能是Web服务器和APP服务器之间的网络出现问题。Example 12中的日志显示plugin把请求写入了请求然后等待响应。在这个例子中，plugin设置了连接超时为10秒，所以10秒过后还没有响应，plugin就会把这台应用服务器标记为宕机。

> Example 12 Marking a server down after time out	[Thu Jul 07 15:09:48 2005] 00000ed4 00000ffc - TRACE: lib_htrequest: htrequestWrite: Writing the request content	[Thu Jul 07 15:09:48 2005] 00000ed4 00000ffc - TRACE: ws_common: websphereExecute: Wrote the request; reading the response	[Thu Jul 07 15:09:48 2005] 00000ed4 00000ffc - TRACE: lib_htresponse: htresponseRead: Reading the response: 4cf1504	[Thu Jul 07 15:09:58 2005] 00000ed4 00000ffc - TRACE: lib_htresponse: htresponseSetError: Setting the error |1|	[Thu Jul 07 15:09:58 2005] 00000ed4 00000ffc - ERROR: ws_common: websphereExecute: Failed to read from a new stream; App Server may have gone down during read

某些情况下，成功建立连接并且写入请求之后，应用服务器宕机了，但是plugin无法检测出来。这种奇怪的现象一般是因为其中一台服务器是Windows另一台是Unix。

Example 13展示了一个正常收到响应的例子。plugin写入请求然后等待响应，7秒钟后收到响应。

> Example 13 A request that responds	[Thu Jul 07 14:27:00 2005] 00000a20 00000f80 - TRACE: lib_htrequest:	htrequestWrite: Writing the request content	[Thu Jul 07 14:27:00 2005] 00000a20 00000f80 - TRACE: ws_common:	websphereExecute: Wrote the request; reading the response	[Thu Jul 07 14:27:00 2005] 00000a20 00000f80 - TRACE: 	lib_htresponse: htresponseRead: Reading the response: 5397cc	[Thu Jul 07 14:27:07 2005] 00000a20 00000f80 - TRACE: HTTP/1.1 200 OK

Example 14展示了这种情况：plugin写入请求然后等待响应，但是5分钟后下一个请求已经到达了，上个请求的响应还没有收到。

> Example 14 A request that never responds	[Thu Jul 07 14:22:20 2005] 00000a20 00000f88 - TRACE: ws_common: websphereExecute: Wrote the request; reading the response	[Thu Jul 07 14:22:20 2005] 00000a20 00000f88 - TRACE: lib_htresponse: htresponseRead: Reading the response: 4cf1504	[Thu Jul 07 14:27:00 2005] 00000a20 00000f80 - TRACE: lib_util: parseHostHeader: Host: 'localhost', port 80

这种时候，你可以使用网络协议分析器去检测数据包在离开Web服务器后发生了什么事情。也可以叫`iptrace`。网络协议分析器是一种专业性强的工具，一般来说即时网络比较空闲也会产生大量的数据。如果你没有足够的经验，也可以请网络工程师来帮忙抓取。

大多数表面上跟plugin相关的问题实际上都是配置问题和网络问题。除了均衡负载这个特性之外，plugin实际上是一个用来抓取请求，转换URL然后找到匹配，然后把匹配好的请求转发到应用服务器的简单组件。


<h4 id="problem3">Problem: Sessions are being lost</h4>

如果在集群环境中出现了会话数据丢失的情况，那么plugin有可能是问题的根源。

##### 需要收集的数据

- Plugin trace
- Web服务器上的Plugin日志

##### 如何检查

Plugin trace可以给我们展示处理会话亲缘性的详细过程。WAS自带的示例应用程序就可以测试会话亲缘性。下面的URL可以展示WAS正在运行而且设置了一个会话。

	http://servername/HelloHTML.jsp

Example 15中的日志片段展示了会话的创建过程。plugin试图找到跟session相关的cookie，但是没有找到，所以它使用round-robin负载均衡算法找到了一台服务器去创建。

> Example 15 Creating a session

        [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: ws_common: websphereWriteRequestReadResponse: Enter
        [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: ws_common: websphereHandleSessionAffinity: Checking for session affinity [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: ws_common: websphereHandleSessionAffinity: Checking the SSL session id
        [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: lib_htrequest:
        htrequestGetCookieValue: Looking for cookie: 'SSLJSESSION'
        [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: lib_htrequest:
        htrequestGetCookieValue: No cookie found for: 'SSLJSESSION'
        [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: ws_common:
        websphereHandleSessionAffinity: Checking the cookie affinity: JSESSIONID
        [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: lib_htrequest:
        htrequestGetCookieValue: Looking for cookie: 'JSESSIONID'
        [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: lib_htrequest:
        htrequestGetCookieValue: No cookie found for: 'JSESSIONID'
        [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: ws_common: websphereHandleSessionAffinity: Checking the url rewrite affinity: jsessionid [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: ws_common: websphereParseSessionID: Parsing session id from '/HelloHTML.jsp'
        [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: ws_common: websphereParseSessionID: Failed to parse session id
        [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: ws_common: websphereHandleSessionAffinity: Bypassing check for partitionID cookie affinity. No stored partition table.
        [Mon Jun 27 14:48:33 2005] 00000798 00000e00 - TRACE: ws_server_group: serverGroupNextRoundRobinServer: Round Robin load balancing
        ...
        [Mon Jun 27 14:51:36 2005] 00000798 00000e18 - TRACE: ws_server_group: serverGroupIncrementConnectionCount: Server kll6571Node01_server2 picked, pendingConnectionCount 1 totalConnectionsCount 5.
下面的Example 16展示了第二个请求处理的过程。因为已经有一个session ID了，所以这一次请求被发送到了Example 15中创建session的那台机器上，也就是`kll6571Node01_server2 `。

> Example 16 Processing session affinity
	
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_common: websphereHandleSessionAffinity: Checking for session affinity [Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_common: websphereHandleSessionAffinity: Checking the SSL session id
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: lib_htrequest:
	htrequestGetCookieValue: Looking for cookie: 'SSLJSESSION'
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: lib_htrequest:
	htrequestGetCookieValue: No cookie found for: 'SSLJSESSION'
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_common: websphereHandleSessionAffinity: Checking the cookie affinity: JSESSIONID [Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: lib_htrequest: htrequestGetCookieValue: Looking for cookie: 'JSESSIONID'
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: lib_htrequest: htrequestGetCookieValue: name='JSESSIONID', value='00004RRFkLCkLGWVw-37Cd-mEN7:10ig7jfqi'
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_common: websphereParseCloneID: Parsing clone ids from '00004RRFkLCkLGWVw-37Cd-mEN7:10ig7jfqi'
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_common: websphereParseCloneID: Adding clone id '10ig7jfqi'
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_common: websphereParseCloneID: Returning list of clone ids
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_server_group: serverGroupFindClone: Looking for clone
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_server_group: serverGroupGetFirstPrimaryServer: getting the first primary server
	￼￼￼￼￼￼￼￼￼￼WebSphere Application Server V6: Web Server Plug-in Problem Determination 23
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_server_group:
	serverGroupFindClone: Comparing curCloneID '10ig7jfqi' to server clone id
	'10ig7jdvd'
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_server_group:
	serverGroupGetNextPrimaryServer: getting the next primary server
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_server_group: serverGroupFindClone: Comparing curCloneID '10ig7jfqi' to server clone id '10ig7jfqi'
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_server_group: serverGroupFindClone: Match for clone 'kll6571Node01_server2'
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_server: serverHasReachedMaxConnections: currentConnectionsCount 0, maxConnectionsCount -1.
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - STATS: ws_server_group: serverGroupCheckServerStatus: Checking status of kll6571Node01_server2, ignoreWeights 1, markedDown 0, retryNow 0, wlbAllows -1 reachedMaxConnectionsLimit 0
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_server: serverHasReachedMaxConnections: currentConnectionsCount 0, maxConnectionsCount -1.
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_server_group: serverGroupIncrementConnectionCount: Server kll6571Node01_server2 picked, pendingConnectionCount 1 totalConnectionsCount 7.
	[Mon Jun 27 14:51:51 2005] 00000798 00000e00 - TRACE: ws_common: websphereHandleSessionAffinity: Setting server to kll6571Node01_server2

查看plugin trace日志就可以知道会话亲缘性是否正常工作，plugin有没有把会话转发到正确的服务器。

如果会话亲缘性很明显没有正常工作，请检查WAS的配置，以确定会话管理的配置参数是否正确设定。同时，检查plugin-cfg.xml文件以确定`CloneID`参数是否存在。

Figure 6展示了会话管理参数的配置页面，按照此路径到达这个配置页：Application servers → servername → Web container settings → Session management

有三种会话机制可以配置。

- 启用SSL ID跟踪，只有为应用程序使用SSL加密的时候才可以使用。
- 启用cookies，这是最通用的方法
- 启用URL重写，这种方法很少使用，因为它需要对每个URL都进行重写，对性能会有较大影响

要启用会话支持，可以选择一个或多个机制。其他参数是用来设置会话能持续多久，和其他目的会话配置。

![](http://dellyqiao.qiniudn.com/2015/04/11/figure6.png)

与会话相关的错误信息只会出现在plugin trace里。

Example 17与Example 16类似，不同的是创建会话的那台主机已经宕机。这时plugin会选择另一台集群中的服务器，但是宕掉的机器产生的会话数据将会丢失。

> Example 17 Session lost after application server goes down
	
	[Thu Jul 07 15:55:59 2005] 00000bcc 000010c0 - TRACE: ws_server_group: serverGroupFindClone: Match for clone 'm23vnx60Craig01_server02'
	...
	[Thu Jul 07 15:55:59 2005] 00000bcc 000010c0 - TRACE: ws_server_group: lockedServerGroupUseServer: Server m23vnx60Craig01_server02 picked, weight 0. [Thu Jul 07 15:55:59 2005] 00000bcc 000010c0 - TRACE: ws_common: websphereFindTransport: Finding the transport
	[Thu Jul 07 15:55:59 2005] 00000bcc 000010c0 - TRACE: ws_common: websphereFindTransport: Setting the transport(case 2): m23vnx60 on port 19082 [Thu Jul 07 15:55:59 2005] 00000bcc 000010c0 - TRACE: ws_common: websphereExecute: Executing the transaction with the app server
	[Thu Jul 07 15:55:59 2005] 00000bcc 000010c0 - TRACE: ws_common: websphereGetStream: Getting the stream to the app server
	[Thu Jul 07 15:55:59 2005] 00000bcc 000010c0 - TRACE: ws_transport: transportStreamDequeue: Checking for existing stream from the queue
	[Thu Jul 07 15:56:00 2005] 00000bcc 000010c0 - ERROR: ws_common: websphereGetStream: Failed to connect to app server on host 'm23vnx60', OS err=10061
	[Thu Jul 07 15:56:00 2005] 00000bcc 000010c0 - TRACE: ws_common: websphereGetStream: socket 6628 closed - failed to connect
	[Thu Jul 07 15:56:00 2005] 00000bcc 000010c0 - ERROR: ws_common: websphereExecute: Failed to create the stream
	[Thu Jul 07 15:56:00 2005] 00000bcc 000010c0 - ERROR: ws_server: serverSetFailoverStatus: Marking m23vnx60Craig01_server02 down
	[Thu Jul 07 15:56:00 2005] 00000bcc 000010c0 - STATS: ws_server: serverSetFailoverStatus: Server m23vnx60Craig01_server02 : pendingConnections 0 failedConnections 1 affinityConnections 1 totalConnections 0.
	[Thu Jul 07 15:56:00 2005] 00000bcc 000010c0 - ERROR: ws_common: websphereHandleRequest: Failed to execute the transaction to 'm23vnx60Craig01_server02'on host 'm23vnx60'; will try another one
	...
	[Thu Jul 07 15:56:00 2005] 00000bcc 000010c0 - TRACE: ws_server_group: lockedServerGroupUseServer: Server kll6571Node01_server01 picked, weight 0. [Thu Jul 07 15:56:00 2005] 00000bcc 000010c0 - TRACE: ws_common: websphereFindTransport: Finding the transport
	[Thu Jul 07 15:56:00 2005] 00000bcc 000010c0 - TRACE: ws_common: websphereFindTransport: Setting the transport(case 2): kll6571 on port 9082

如果从plugin的角度来看，很明显会话亲缘性工作正常，那问题很可能出现在应用上。这时请跟开发团队去确定应用程序有正确地处理会话。

Example 18展示了这样的情况：应用程序需要会话，并且采用了cookie机制，但是客户端浏览器不支持cookie。由于无法保存session ID，导致plugin无法维护会话亲缘性。

> Example 18 Session failure as cookies are not allowed
	
	[Thu Jul 07 15:59:04 2005] 00000978 00000ff4 - TRACE: ws_common: websphereHandleSessionAffinity: Checking for session affinity [Thu Jul 07 15:59:04 2005] 00000978 00000ff4 - TRACE: ws_common: websphereHandleSessionAffinity: Checking the SSL session id
	[Thu Jul 07 15:59:04 2005] 00000978 00000ff4 - TRACE: lib_htrequest: htrequestGetCookieValue: Looking for cookie: 'SSLJSESSION'
	[Thu Jul 07 15:59:04 2005] 00000978 00000ff4 - TRACE: lib_htrequest: htrequestGetCookieValue: No cookie found for: 'SSLJSESSION'
	[Thu Jul 07 15:59:04 2005] 00000978 00000ff4 - TRACE: ws_common: websphereHandleSessionAffinity: Checking the cookie affinity: JSESSIONID
	[Thu Jul 07 15:59:04 2005] 00000978 00000ff4 - TRACE: lib_htrequest: htrequestGetCookieValue: Looking for cookie: 'JSESSIONID'
	[Thu Jul 07 15:59:04 2005] 00000978 00000ff4 - TRACE: lib_htrequest: htrequestGetCookieValue: No cookie found for: 'JSESSIONID'
	[Thu Jul 07 15:59:04 2005] 00000978 00000ff4 - TRACE: ws_common: websphereHandleSessionAffinity: Checking the url rewrite affinity: jsessionid [Thu Jul 07 15:59:04 2005] 00000978 00000ff4 - TRACE: ws_common: websphereParseSessionID: Parsing session id from '/HelloHTML.jsp'
	[Thu Jul 07 15:59:04 2005] 00000978 00000ff4 - TRACE: ws_common: websphereParseSessionID: Failed to parse session id

查阅下面的连接获取更多有关WAS集群中会话管理的信息：[http://publib.boulder.ibm.com/infocenter/wasinfo/v6r0/index.jsp?topic=/com.ibm.websphere.nd.doc/info/ae/ae/crun_srvgrp.html](http://publib.boulder.ibm.com/infocenter/wasinfo/v6r0/index.jsp?topic=/com.ibm.websphere.nd.doc/info/ae/ae/crun_srvgrp.html)


<h4 id="problem4">Problem: The application works intermittently</h4>

如果在集群环境下，系统发生定期的的反常的现象，可能是plugin有问题。

##### 需要收集的数据

- Web服务器日志
- plugin日志
- WAS日志

如果依然不能定位问题，你需要做一个plugin trace。

##### 如何检查

时好时坏的现象可能是因为plugin没有检测到集群中的某台WAS服务器已经宕机，依然把请求转发到那些主机上。

如果应用服务器是宕机的（服务器进程处于停止状态），plugin没办法与它创建TCP/IP连接，也不会把请求发过去。因此在这种情况下，plugin会把这台服务器标记为宕机。

但是，如果应用服务器挂起（hung），那么这台服务器依然可以接收请求，只是不能做出响应。这种情况下，plugin不会把这台服务器标记为宕机，除非从操作系统层面认为连接超时。具体超时的时间依操作系统而定，可能长达十分钟。当plugin标记这台服务器宕机之后，他还会在某些时间点再次检测这台服务器是否依然宕机，这时服务器还是可以接收请求，因此plugin会再次标记它为已启动，直到连接从操作系统层面再次超时。

如果应用服务器没有挂起而是处理速度慢，当应用服务器上可用的的TCP/IP连接被用完时，plugin就无法建立新的连接了，就会标记它为宕机。

如果是应用本身有问题，因为应用服务器能接收请求并且做出响应（即使是失败的响应），因此plugin不会认为应用所在的服务器宕机。

> Note: 上面提到的所有情况下，当你直接访问应用服务器的时候都是无法获得响应的，例如：
> http://servername:9080/snoop
> 在集群环境中，你需要单独访问每台服务器以确定哪台有问题。更多WAS排错的方法和信息请查阅此处：http://www.redbooks.ibm.com/redpapers/pdfs/redp4073.pdf

这个部分所提到的问题都是跟应用服务器相关的，所以你需要解决应用服务器的问题。然而，我们可以通过修改plugin的参数去控制plugin如何响应time-out的情况，从而最大限度地减少问题造成的影响。下面介绍一下相关的参数：

- RetryInterval

	这个参数确定在plugin标记某台服务器为宕机状态之后多久再一次检查服务器的状态。

- ServerIOTimeout

	`Server`元素中的这个ServerIOTimeout参数允许我们设定plugin发送请求到接收响应这段时间的超时时间的值，单位是秒。如果没有设定这个参数，plugin会使用阻塞I/O方式把请求写入到应用服务器，然后从应用服务器读取响应，直到这个TCP连接超时。
	
		<Server Name="server1" ServerIOTimeout=300>

	在这个配置中，在TCP连接超时之前，plugin会等待300秒（也就是5分钟）。设置一个合理的值可以使plugin尽可能快速地把请求切换到可用的服务器上。同时也注意，应用程序处理请求的时候可能需要几分钟，如果超时时间设置得太短可能会导致plugin给客户端发送错误的响应。
	
	> Note: 运行在Solaris操作系统上的plugin会忽略ServerIOTimeout参数。

- ExtendedHandshake

	如果在Web服务器plugin和应用服务器之间有代理防火墙，你应该配置这个参数。这个参数强制plugin去做更广泛的检查以确定应用服务器是否宕机。

- MaxConnections

	这个参数限定并行发送到应用服务器的连接数，从而避免plugin发送过量的请求拖慢速度。

- ConnectTimeout

	`Server`元素中的这个ConnectTimeout参数使plugin可以跟应用服务器发起一个非阻塞连接。当plugin无法连接到目标服务器因而无法确定端口是否可用的时候，非阻塞连接非常有用。
	
	如果不设定这个参数或者设置值为0，plugin会从所在的服务器发起一个阻塞连接，这个连接的超时时间取决于操作系统，一旦超时plugin会标记那台服务器宕机。如果设置的数字大于0，plugin会等待对应的秒数。如果在这个超时时间之后，链接始终没有建立成功，plugin会标记宕机，尝试集群中的其他成员。
	
	***译者注：这个参数与ServerIOTimeout的区别是，这个超时时间是指建立连接的超时时间，关注点在创建连接本身的这个过程；而ServerIOTimeout是配置整个处理过程的超时时间，也就是从plugin发送请求开始算起，不管连接有没有建立起来，它只关注是否在特定的时间内收到了响应***

如果可能的话，尽量使用WAS Console去配置这些参数，因为一旦重新生成插件，手工修改的参数就不存在了。我们可以用Console设置`RetryInterval`这个参数，跳转的路径为：Web servers → webserver → Plug-in properties → Request routing，如Figure 7。

![](http://dellyqiao.qiniudn.com/2015/04/11/figure7.png)

其他所有的参数都得修改plugin-cfg.xml手工配置。所以每次重新生成plugin之后，记得手工修改一下。更多plugin属性的配置请参阅此链接：[http://publib.boulder.ibm.com/infocenter/wasinfo/v6r0/index.jsp?topic=/com.ibm.websphere.express.doc/info/exp/ae/rwsv_plugincfg.html](http://publib.boulder.ibm.com/infocenter/wasinfo/v6r0/index.jsp?topic=/com.ibm.websphere.express.doc/info/exp/ae/rwsv_plugincfg.html)

<h4 id="problem5">Problem: Application load is not being evenly distributed</h4>

负载分部不均匀可能也是由plugin引起的。

##### 需要收集的数据

- Plugin trace
- Plugin日志

手机plugin日志即可得到trace，因为trace是写入到plugin日志中的。你需要分析trace以确定每台服务器得到多少请求。

##### 如何检查

每当plugin选择一台服务器发送请求时都会在日志里写入一条类似下面这样的跟踪信息：

TRACE: ws_server_group: serverGroupIncrementConnectionCount: Server servername picked, pendingConnectionCount count totalConnectionsCount count.

你可以手动数一下plugin为集群里的没台应用服务器发送了多少个请求，也可以写一个脚本把日志数据转换一下，然后得到这个数量。（***译者注：如果是Linux系统的话，直接用`grep xxxx plugin.log | wc -l`就可以了***）。如果发现负载分发的时候不均匀，我们需要检查一下所定义的每个集群成员的权重，查看`LoadBalanceWeight`参数即可：

	<Server CloneID=cloneid ConnectTimeout=0 ExtendedHandshake=false
	LoadBalanceWeight=3 MaxConnections=-1 Name=servername
	ServerIOTimeout=0 WaitForContinue=false>

每当plugin把请求发送到某一台应用服务器，那台服务器所对应的计数器的数值会减一。计数器的初始值就是权重的数值。当一台服务器计数器的数值变为0的时候，plugin就不会再往那里转发请求了。当所有服务器的数值都变为0的时候，plugin会充值计数器的数值为权重值。权重的默认值是2。

举个例子，集群里有两台服务器，server1和server2。Server1的权重是2，server2是4。Table 1展示了请求会如何发送到两台服务器上。

![](http://dellyqiao.qiniudn.com/2015/04/11/table1.png)

其他参数的配置也会影响到负载均衡。举例来说，如果某台服务器上的应用运行速度比较慢，那台服务器就很容易到达`MaxConnection`的上限，这时plugin就会停止给它发送请求。如果发生这种情况，我们可以在plugin trace里看到。

会话亲缘性也可能影响到请求被分发到哪台服务器上。如果某台服务器创建了一个会话，那后续的请求当然都会发送到那台服务器上。

如果每台服务器收到的请求数的比值跟权重设置的数字的比值相同，就说明plugin的负载均衡机制工作正常。这时异常的CPU占用率肯定就是由其他原因引发的了，比如服务器上的其他的异常进程，或者应用程序有问题。

如果比值不同，就说明其他因素影响了负载均衡的工作。比如，某台服务器的请求数到达了`MaxConnections`的上限，或者某台服务器被plugin标记为宕机了。

如果你需要更智能的负载均衡方案，比如根据应用服务器的负载或响应时间，你可以使用其他的负载均衡产品，比如WAS Edge组件。

<h4 id="problem6">The next step</h4>

本文中提到的那些问题特征基本上涵盖了可能遇到的大部分情况，然而也有其他可能性导致问题的出现。如果看完了本文的诊断过程还是没有得到真相，强烈建议你看一下这篇WAS问题诊断的文章：http://www.redbooks.ibm.com/redpapers/pdfs/redp4073.pdf。请查看这一章：Classify the problem and determine the root cause，以确定是否问题出在其他组件上。

如果你觉得肯定是plugin的问题，那就重新查看一下你收集到的所有诊断文件，查找跟问题相关但是本文有没有提到的日志记录，查阅支持网站。

如果还是不能定位问题，你需要收集下面列表中提到的“MustGather”文档，然后发送这些文档给IBM进行下一步诊断。

- MustGather for general problems with the Web server plug-in: 

	http://www-1.ibm.com/support/docview.wss?rs=180&uid=swg21174894
- MustGather for problems generating the plug-in configuration file:
	
	http://www-1.ibm.com/support/docview.wss?rs=180&uid=swg21199421􏰀
- MustGather for problems installing the Web server plug-in: 

	http://www-1.ibm.com/support/docview.wss?rs=180&uid=swg21199343


请把你已经做过的诊断过程告诉IBM的支持工程师以减少诊断问题的时间。


----

全文完，接下来会继续翻译 redp4309-WebSphere Application Server V6.1Web container problem determination.pdf