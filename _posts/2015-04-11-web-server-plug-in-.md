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

