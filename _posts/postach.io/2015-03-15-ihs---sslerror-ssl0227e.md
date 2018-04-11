---
layout: post
title: "IHS   SSL证书替换之后无法访问（Error SSL0227E）"
description: ""
category: ["Middleware", "Security"]
---
{% include JB/setup %}


### 出现的问题

今天在为网站替换SSL证书的时候出现了一个问题，服务器是IBM HTTP Server服务器，症状如下：

- 替换之后站点无法访问
- 用ikeyman打开证书查看，证书没有问题
- 服务启动后，HTTPS端口（443）已经正常监听了

<!-- more -->

### 问题分析

如果证书没有问题的话多半就是配置文件的问题了。查看error.log，发现如下错误：

	[Fri Jan 16 14:28:36 2015] [crit] [client 10.64.48.52] [2195808] [2864] SSL0227E: SSL Handshake Failed, Specified label could not be found in the key file. [10.64.48.52:37368 -> 10.64.48.29:443] [14:28:36.457113] 
	[Fri Jan 16 14:28:36 2015] [crit] [client 221.122.53.200] [2195808] [2864] SSL0227E: SSL Handshake Failed, Specified label could not be found in the key file. [221.122.53.200:31042 -> 10.64.48.29:443] [14:28:36.457113] 
	[Fri Jan 16 14:28:36 2015] [crit] [client 10.64.48.51] [2195808] [2864] SSL0227E: SSL Handshake Failed, Specified label could not be found in the key file. [10.64.48.51:46678 -> 10.64.48.29:443] [14:28:36.457113] 


`pecified label could not be found in the key file.` 错误信息里，这句话是关键，指定的label无法在key文件中找到。就是说httpd.conf中指定了一个别名，但是在key.kdb文件中没有。

查看httpd.conf，发现如下配置：


	Keyfile "D:\IBM\HTTPServer\SSL\2015\www.server.com\key.kdb"
	SSLserverCert www.server.com


`SSLserverCert` 这个配置是指定证书文件的名字，因为在key.kdb文件中可以导入多个个人证书，在一个kdb文件里有多个个人证书的情况下，就必须要指定你当前的配置是要用哪个证书去加密会话。


一般在制作CSR的时候，我们习惯用域名来作为Label Name（类似于www.server.com），所以在我们的httpd.conf配置文件中都会用指定`SSLserverCert www.server.com`。但是这次的KDB文件是由其他vendor提供的，在制作CSR的时候它们随便用了一个Label Name，所以就导致找不到证书别名的问题。


### 解决方法

- 直接把`SSLserverCert`这一行删掉。因为我们的kdb文件里只有一个CER证书，删掉是无所谓的。

- 把那个别名改变一下，跟导入是的别名设置成一样的。
	
	如何查看证书导入时候的别名？如下图了：

	![SSL Alias](http://dellyqiao.qiniudn.com/2015/01/16/ssl alias.png/scale)