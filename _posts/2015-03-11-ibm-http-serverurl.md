---
layout: post
title: "在IBM HTTP Server中配置URL重定向"
description: ""
category: "Middleware"
---
{% include JB/setup %}

之前总结在Apache中实现URL重定向跳转的功能，方法如下：


> 比如要想把dellyserver/abc/下所有的资源都重定向到www.baidu.com，我的dellyserver是在DMZ中，这台server无法访问百度的服务器。

<!-- more -->
> 第一步，写一个html文件，就叫test.html吧，放到网站的根目录。test.html只是个空白网页，只需要在<head>标签里加上下面这行JS代码：

> <pre>window.location.href="http://www.baidu.com”</pre>

> 第二步，在httpd.conf中添加配置如下：

	RewriteCond %{HTTP_HOST} ^dellyserver [NC] 

	RewriteRule ^/abc/(.*)$ /test.html [L]

> 这样所有到达http://dellyserver/abc/xxxx的资源都会被定向到test.html，当test.html下载到本地的时候，浏览器会执行html文件中的JS代码：<pre>window.location.href="http://www.baidu.com"</pre>，这样浏览器会自动给http://www.baidu.com发送请求并且获得响应，费了一番功夫但是也确实达到了重定向的效果。

	  

IHS是基于Apache的，按理说这种方法也应该能工作的，但是起作用的情况仅限静态网站的情况。

如果IHS通过Plugin进行URI的处理，那么用上面的方法并不会生效，而是会直接跳出500错误。因为IHS会先通过plugin匹配用户的URI请求，然后再搜索DocumentRoot下地文件去匹配请求。

如果请求abc.jsp，如果plugin无法处理对应的请求，IHS就会直接返回500错误，因为jsp只能被WAS处理，DocumentRoot下的文件全部都是静态html文件。

  


那么应该如何配置才能让IHS首先去适配rewrite模块填写的规则呢？如下：

	RewriteRule ^/abc/(.*)$ /test.html [L,R=301]*



R=301 代表/abc/下所有的资源都永久转移到新的位置了，这样IHS就会去直接执行这条规则而不是去尝试plugin匹配了。  

