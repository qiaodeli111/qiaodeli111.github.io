---
layout: post
title: 解决IHS/Plugin/WAS环境中请求转发失败的问题（500错误）
category: "Middleware"
date: 2015-02-09
---

这几天遇到了一个IHS到WAS请求转发失败的问题，所以就打算把常见的几种错误和排错的经验写出来了。

文章内容仅适用于IHS和WAS分别安装在不同服务器上的情况，文章里提到的一些错误在其他情况下也许根本就不会发生。

<!-- more -->


## 本例中环境的配置：

IHS和WAS分别装在两台服务器上，两台服务器分别位于不同的DMZ，DMZ之间有防火墙，在防火墙上只开通了必要的端口。

在本例中，公网用户通过公网IP(public_ip)，端口80和443访问IHS服务器，IHS服务器(IP地址为：ihs_server)上安装了Plugin，plugin通过端口9081把网站的请求转发到WAS服务器(IP地址为：was_server)。假设要请求的资源在WAS服务器上，叫abc.jsp.

  

## 出现的问题：

从公网测试：

http://public_ip/abc.jsp — abc.jsp可以正常显示

https://public_ip/abc.jsp — 500 Internal Server Error

  


从IHS服务器上测试：

http://ihs_server/abc.jsp — abc.jsp可以正常显示

https://ihs_server/abc.jsp — 500 Internal Server Error

  


从IHS和WAS服务器上分别测试：

http://was_server:9081/abc.jsp -- abc.jsp可以正常显示

  


## Troubleshooting和问题分析


### 1）首先看plugin_cfg.log，错误信息如下：

```
[Wed Dec 03 15:40:13 2014] 00000dbc 00000e78 - ERROR: ws_common: websphereGetStream: Connect timeout fired

[Wed Dec 03 15:40:18 2014] 00000dbc 00000e78 - ERROR: ws_common: websphereGetStream: Connect timeout fired

[Wed Dec 03 15:40:18 2014] 00000dbc 00000e78 - ERROR: ws_common: websphereExecute: Failed to create the stream

[Wed Dec 03 15:40:18 2014] 00000dbc 00000e78 - ERROR: ws_common: websphereHandleRequest: Failed to execute the transaction to ‘wasserverNode01_abc'on host ‘wasserver'; will try another one

[Wed Dec 03 15:40:23 2014] 00000dbc 00000e78 - ERROR: ws_common: websphereGetStream: Connect timeout fired

[Wed Dec 03 15:40:28 2014] 00000dbc 00000e78 - ERROR: ws_common: websphereGetStream: Connect timeout fired

[Wed Dec 03 15:40:28 2014] 00000dbc 00000e78 - ERROR: ws_common: websphereExecute: Failed to create the stream

[Wed Dec 03 15:40:28 2014] 00000dbc 00000e78 - ERROR: ws_common: websphereHandleRequest: Failed to execute the transaction to ‘wasserverNode01_abc'on host ‘wasserver'; will try another one

[Wed Dec 03 15:40:28 2014] 00000dbc 00000e78 - ERROR: ws_common: websphereWriteRequestReadResponse: Failed to find an app server to handle this request

[Wed Dec 03 15:40:28 2014] 00000dbc 00000e78 - ERROR: ESI: getResponse: failed to get response: rc = 2

[Wed Dec 03 15:40:28 2014] 00000dbc 00000e78 - ERROR: ws_common: websphereHandleRequest: Failed to handle request
```
  


这段错误的LOG完全给不出有用的信息，只是在说无法建立连接，导致请求处理失败。

奇怪的地方是http没问题但是https就不行。

  


### 2）继续检查其他所有的Log，包括IHS的error log——没有任何错误信息，WAS的JVM log——用https做的访问请求完全没有任何输出，意味着请求完全没有到达WAS应用。


### 3）继续怀疑是plugin config文件的问题，但是试过很多更改，全部都是一样的效果。

  
怎么办？还是怀疑问题出在plugin的转发过程上。想起来plugin的Log level是可以更改的，我们所设置的leve是ERROR，只有在plugin转发出现ERROR信息的时候才会保存到plugin_cfg.log文件内。现在试着把完整地跟踪信息输出。


更改plugin_cfg.xml： 找到这行配置：_LogLevel_=“Error”， 把Error改成Trace即可。可以等1分钟自动重新加载plugin配置文件或者直接重启IHS服务立刻生效。


果然发现问题了，在plugin log里有这么一段：

  

```
[Wed Dec 03 17:17:07 2014] 00001220 000012a4 - TRACE: ws_server_group: lockedServerGroupUseServer: Server wasserver_abc picked, weight 2.

[Wed Dec 03 17:17:07 2014] 00001220 000012a4 - TRACE: ws_common: websphereFindTransport: Finding the transport

[Wed Dec 03 17:17:07 2014] 00001220 000012a4 - DETAIL: ws_common: websphereFindTransport: Setting the transport(case 1): wasserver on port 9444

[Wed Dec 03 17:17:07 2014] 00001220 000012a4 - TRACE: ws_common: websphereExecute: Executing the transaction with the app server reqInfo is OKuseExistingStream=0, client->stream=00000000

[Wed Dec 03 17:17:07 2014] 00001220 000012a4 - DEBUG: ws_common: websphereGetStream: Getting the stream to the app server

[Wed Dec 03 17:17:07 2014] 00001220 000012a4 - TRACE: ws_transport: transportStreamDequeue: Checking for existing stream from the queue

[Wed Dec 03 17:17:07 2014] 00001220 000012a4 - TRACE: ws_common: websphereGetStream: Have a connect timeout of 5; Setting socket to not block for the connect

[Wed Dec 03 17:17:12 2014] 00001220 000012a4 - TRACE: errno 0

[Wed Dec 03 17:17:12 2014] 00001220 000012a4 - TRACE: RET 0

[Wed Dec 03 17:17:12 2014] 00001220 000012a4 - TRACE: READ SET 0

[Wed Dec 03 17:17:12 2014] 00001220 000012a4 - TRACE: WRITE SET 0

[Wed Dec 03 17:17:12 2014] 00001220 000012a4 - TRACE: EXCEPT SET 0

[Wed Dec 03 17:17:12 2014] 00001220 000012a4 - ERROR: ws_common: websphereGetStream: Connect timeout fired
```
  

实际上看到这一行：Setting the transport(case 1): wasserver on port 9444 就大概明白了，IHS在进行请求转发的时候使用的不是9081，而是9444，9444是wc_default_secure端口，也就是WAS服务器容器的安全传输端口。而9444端口在IHS和WAS之间并没有打开。

  
看来在处理https请求的时候，IHS会尝试使用安全端口吧请求转发到WAS。


所以现在的问题变成了如何让IHS利用9081端口而不是9444端口？



## 问题解决

在plugin_cfg.xml配置文件中，有一行关于kdb文件的配置信息，里面写明了所使用的plugin.kdb文件放置的路径。删掉这行配置，或者删掉对应的kdb文件就好了。


KDB文件是IBM的用来存放安全证书的文件，安全证书用来给session加密。下一次当IHS找不到安全证书的时候自然就会退而求其次使用没有安全加密的端口9081了。
