---
layout: post
title: "如何在IBM WAS上配置主从集群"
description: ""
category: "Middleware"
---
{% include JB/setup %}

如题，如何在IBM WAS（WebSphere Application Server）上配置主从集群呢？

一个典型的基于IBM中间件的生产环境网站系统包含这几个部分：

1. 前端的LoadBalancer（软件或者硬件）
2. IBM HTTP Server集群（处理静态内容，通过前端的LoadBalaner分发请求到多个节点上）
3. IBM WebSphere Plugin（安装在IBM HTTP Server所在的主机上，用来把动态请求转发到后端的IBM WebSphere Application Server服务器集群上）
4. WAS服务器
5. 后端的DB服务器

WAS集群实际上分两种情况，一种是使用了ND（Network Deployment）版本的，另一种是使用了Base版的。我们分别来探讨。

<!-- more -->


--------

### WAS ND版 ###

WAS ND版创建集群比较方便，在管理控制台的Console上就有相关功能（Click Servers -> Clusters -> WebSphere application server clusters, 点击`新建`.）根据提示一路向下就能创建好集群。

默认情况下，集群创建好之后，每个集群成员的权重都是一样的，外部请求会被Plugin以Round Robin方式平均地把请求转发到后端服务器上。所以我们可以想到的第一个办法就是***通过修改对应服务器的权重达到主从的目的***。

具体操作如下：

1. 登录到管理控制台，定位到集群配置界面（Servers -> Clusters -> WebSphere application server clusters.
2. 选择对应的集群，点击`成员`即可对集群成员的权重进行更改。
3. 将standby成员的权重设置成 `0`，这样所有的请求都不会跑到它这边，除非其他集群成员无法处理请求（比如宕机）。
4. **重新生成plugin-cfg配置文件，并且替换到WEB服务器上。**

第二个方法是**配置应用服务器的角色（role）**。

默认情况下，所有应用服务器的角色都是“Primary”，所以我们需要把从成员的角色设定为“Backup”。操作方法如下：

1. 找到要设置为backup的服务器，进入配置界面，点击Additional Properties -> web server plug-in properties

	![http://dellyqiao.qiniudn.com/2016/05/10/serverrole_1.png](http://dellyqiao.qiniudn.com/2016/05/10/serverrole_1.png/scale)

2. 这里可以配置服务器的角色，进而plugin就可以通过plugin-cfg配置文件知道服务器的角色。

	![http://dellyqiao.qiniudn.com/2016/05/10/serverrole_2.png](http://dellyqiao.qiniudn.com/2016/05/10/serverrole_2.png/scale)

3. **重新生成plugin-cfg配置文件，并且替换到WEB服务器上。**

--------

### WAS Base版 ###

在WAS Base版中上面的做法是没用的。因为Base版没有DMGR，没有统一的管理，即使有服务器集群也都是通过多个单点服务器的堆叠而成的。Plugin-cfg配置文件也必须通过IBM提供的工具进行手动merge。以上ND提到的方法拿到Base版本里就有了一些限制：

- Base版由于没有集群的概念，所以也没有权重这个东西，所以第一种办法不可能实现。
- Base版本也可以配置应用服务器的角色。但是配置过后还需要手动merge plugin-cfg配置文件，而merge工具会平等地看待所有的plugin-cfg配置文件，所以最终得到的配置文件并不会体现出每个服务器的规则是主还是从。

> 下图为Base版本集群的架构：
> 
![http://7rfgqc.com1.z0.glb.clouddn.com/2016/05/10/2.png](http://7rfgqc.com1.z0.glb.clouddn.com/2016/05/10/2.png/scale)



**但是隐约觉得第二种方法是有可能实现的，因为最终哪个节点对应什么规则实际上是由WebSphere Plug-in来决定的，而WAS版本的差异只在于WAS而已，plugin没有任何差异。所以，只要plugin可以把请求按照主从的方式转发到后台的WAS服务器，那么我们的主从集群就有可能实现。**

根据以上猜想，我手动检查了ND版本实现了主从集群对应的的plugin-cfg配置文件，并且找到了控制主从集群的关键属性：

	<ServerCluster CloneSeparatorChange="false" GetDWLMTable="false>
	......
	<BackupServers>
    	<Server Name="app01Node01_AppSrv_1"/>
	</BackupServers>
    <PrimaryServers>
		<Server Name="app02Node01_AppSrv_0"/>
	</PrimaryServers>
	......
	</ServerCluster>

非主从集群的配置文件会是如下的情形：

	<ServerCluster CloneSeparatorChange="false" GetDWLMTable="false>
	......
	<PrimaryServers>
    	<Server Name="app01Node01_AppSrv_1"/> 
		<Server Name="app02Node01_AppSrv_0"/>
	</PrimaryServers>
	......
	</ServerCluster>

所以，对于WAS Base版本，如果希望配置WAS服务器集群为主从集群，我们就必须要手动配置：

1. 登录到每个应用服务器对应的管理控制台，生成plugin-cfg.xml配置文件。
2. 使用`<WAS HOME>/pluginCfgMerge`工具把多个plugin-cfg配置文件合并为一个。
3. 手动修改生成后的plugin-cfg文件，把主成员放到`<PrimaryServers>`标签中，把从成员放到`<BackupServers>`标签中。
4. 把修改后的plugin-cfg文件替换到所有的WEB服务器上。


--------

全文完。