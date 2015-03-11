---
layout: post
title: "利用Apache服务器实现URL跳转"
description: ""
category: "Middleware"
---
{% include JB/setup %}


此文记录一下如何利用Apache服务器进行URL跳转。开发人员可能会说用JavaScript或者后台代码可以很容易实现，但是不管是什么样的代码，都需要首先载入页面，之后才能实现跳转功能，体验并不是很好，如果在Apache服务器上设置好的话，由于服务器的配置已经预先读取到内存里了，所以当请求到达Apache的时候直接就可以进行相应地跳转，迅速可靠。

  


  


Apache的URL重定向是利用rewrite_module实现的，这个模块功能十分强大，作为一个实用主义者我只取需要用到的功能。

<!-- more -->
  


**注意**：如果想要做的跳转是跨服务器的跳转，那么Apache所在的服务器必须得能访问目标服务器。比如说Apache在服务器A上，要想把abc.com/abc重定向到www.baidu.com，那么从服务器A上至少得能访问到www.baidu.com，否则跳转是不会成功的。所以说如果服务器在DMZ里，四面都有防火墙，那就请参照本文最后面的配置方法。

  


如何配置此模块：

1）找到这个模块的配置，把前面的注释符号去掉。

	LoadModule rewrite_module modules/mod_rewrite.so

2）在`<virtualhost>` 的配置中，添加如下配置：

	RewriteEngine On

配置完成。

注意：RewriteEngine必须要在virtualhost里启用，否则不会生效。

  


在这行配置下面就可以写跳转的规则了。下面是常见的几种情况：

#### 1）单个指定URL的跳转。

比如要把/abc.html重定向到/def.html：

	Redirect /abc.thml /def.html

  


#### 2）有相同特征的域名的跳转。

比如说把所有/abc/之下的所有页面都重定向到/def/1.html：

	RewriteRule ^/abc/(.*)$ /def/1.html

rewrite_module支持正则表达式，^表示URI的开头，.表示任意字符，*表示出现N次，$表示结尾。

  


#### 3）还有一种情况，域名更换，但是所有资源的位置没有改变。

比如，访问http://dellyserver/abc/abcd.html的时候跳转到http://abc.dellyserver/abcd.html，是要根据所请求的资源动态跳转，而不是跳到一个特定的页面。


	RewriteCond %{HTTP_HOST} ^dellyserver [NC]
	##声明一下请求的URL的hostname是dellyserver

	RewriteCond %{REQUEST_URI} ^/abc/
	##声明一下请求的URI是/abc/xxxxxx

	RewriteRule ^/abc/(.*) <http://abc.dellyserver/$1> [R=permanent,L]
	##在以上两个条件都符合的时候，执行这个跳转，$1代表的是http://dellyserver/abc/后面的部分
  


4） 最后一种情况，跨服务器跳转，并且源服务器无法访问目标服务器的情况。

比如要想把dellyserver/abc/下所有的资源都重定向到www.baidu.com，我的dellyserver是在DMZ中，这台server无法访问百度的服务器。

这种时候，根据我目前的知识，唯一能想到的办法应该是借助程序代码了。

  


第一步，写一个html文件，就叫test.html吧，放到网站的根目录。test.html只是个空白网页，只需要在<head>标签里加上下面这行JS代码：

	<pre>window.location.href="http://www.baidu.com”</pre>

  


第二步，在httpd.conf中添加配置如下：

	RewriteCond %{HTTP_HOST} ^dellyserver [NC]

	RewriteRule ^/abc/(.*)$ /test.html [L]

这样所有到达http://dellyserver/abc/xxxx的资源都会被定向到test.html，当test.html下载到本地的时候，浏览器会执行html文件中的JS代码：<pre>window.location.href="http://www.baidu.com"</pre>，这样浏览器会自动给http://www.baidu.com发送请求并且获得响应，费了一番功夫但是也确实达到了重定向的效果。  


  


就这些了，rewrite_module非常强大，有更多需求的同学可以继续深入研究下这个东西。希望我本文能起到抛砖引玉的作用吧。
