---
layout: post
title: "Disable RC4 Cipher for IIS/Apache/IHS"
description: ""
category: ["security", "middleware"]
---
{% include JB/setup %}

最近多家安全网站曝出网站系统使用TLS安全协议时如果使用RC4 Cipher会有安全问题，如下文所述：

[]()

对于常见的Web服务器来说，要修复这个安全漏洞，那就禁用RC4 Cipher。下面列出几种服务器中禁用RC4 Cipher的方法：

<!-- more -->

### 1) IIS

IIS服务器使用哪种安全协议，安全协议使用哪种密钥都是需要在注册表里去配置的。RC4 Cipher也不例外，在Windows Server 2003中的配置方法如下：



### 2) IHS

IHS即IBM HTTP Server，所有的配置都在httpd.conf中，所以修改httpd.conf就好了。

默认情况下，IHS是不会启用RC4的，但是由于前几年各界都比较认可RC4，所以一般的IHS服务器都启用了RC4。想要禁用的话删掉对应的配置就好了。



### 3) Apache

如此连接：

