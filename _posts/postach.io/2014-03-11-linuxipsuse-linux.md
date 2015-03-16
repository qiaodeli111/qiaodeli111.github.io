---
layout: post
title: "Linux系统添加虚拟IP（Suse linux为例）"
description: ""
category: "Server"
---
{% include JB/setup %}


之前有提到Windows服务器添加多个虚拟IP的方法，只需要用图形化界面就可以了，但是Linux服务器就没那么简单。所以总结了这么一篇教程，记录一下如何在Linux服务器上添加虚拟IP。

<!-- more -->

1）以Root权限登录，或者登陆以后su - 获取root权限。

2）跳转到Network设置的位置：

	cd /etc/sysconfig/network-scripts

3）查看一下现有的网络适配器有哪些：

	ls ifcfg-eth*

可以看到一个或一些类似于： ifcfg-eth-id-12:34:56:78:90:ab 这样的文件，这一串数字和字母的组合（12:34:56:78:90:ab）就是网卡的MAC码。

4）接下来就编辑这个文件。

打开这个文件：`vi ifcfg-eth-id-12:34:xxxxxx`

可以看到类似于如下的字段：

	BOOTPROTO='static'  
	BROADCAST='192.168.1.1'  
	IPADDR='192.168.1.1'  
	NETMASK='255.255.255.0'  
	NETWORK='192.168.1.1'  
	STARTMODE='onboot'  
	USERCONTROL='no'  
	_nm_name='bus-pci-0000:01:04.0'


接下来要做的，是添加如下的几行：

	IPADDR1='192.168.1.2'  
	NETMASK1='255.255.255.0'  
	LABEL1='0'


5）保存关闭文件，然后重启network服务：

	/etc/init.d/network restart



这样就已经成功了，直接ifconfig查看一下吧，是不是已经有新添加的IP地址了。当然可以通过ping那个新添加的IP来测试一下是否已经成功。

  


其他的Linux distribution可能网卡配置的路径不一样，但是大体原理是一样的，如果有类似的需求，就可以以本文作为参考了。
