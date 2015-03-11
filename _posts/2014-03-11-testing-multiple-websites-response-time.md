---
layout: post
title: "批量检测网站访问速度"
description: ""
category: "Server"
---
{% include JB/setup %}

需求如下：

我有一大堆需要维护的网站，每天早上都需要检查所有的网站是否存在异常然后给management发一份报告。简单来说需要查看每个网站的响应速度是否超过了设定的阈值（比如<10秒）。

如何达成？以前用zabbix监控所有的网站，但是那东西太大型了，我需要一个能在本机运行而且耗费资源很少的工具，所以这里就要用到ab和abs。

<!-- more -->


ab和abs是Apache附带的一个工具，在Apache安装目录的bin文件夹里就能找到，ab用来测试http网站，abs可以测https。想要使用这个工具的话可以下载Apache。如果不想要整个apache目录的话可以只保留bin文件夹，也是没有问题的。

  

我们工作所用的电脑都是windows系统，所以写一个简单的windows脚本来实现批量检查吧。

  


1） 写一个脚本，比如scan.bat，内容如下：

	@echo off
	for /F %%u in (serverlist.txt) do echo %%u >> Report.txt && bin\abs %%u | findstr Time | findstr per | findstr (mean) >> Report.txt >> Report.txt

  


2） 创建一个放置网址的文件 serverlist.txt，把所有需要检测的网站都放进去，比如：

	https://www.baidu.com/
	http://abc.com/index.html

**注意：如果网址没有指定URI，比如http://www.baidu.com/，在网址的最后必须要加上"/"，否则ab和abs无法识别！**


3） 把Apache的bin文件夹拷贝到同目录下，运行scan.bat。


输出结果如下：

	http://www.baidu.com/   
	Time per request: 655.037 [ms] (mean)

	http://www.sina.com/
	Time per request: 633.037 [ms] (mean)

不要问我为什么输出的结果这么难看，不好意思用windows脚本处理字符串实在太蛋疼了。。。。。。

  
如果有Linux系统的话，你可以做的更好，比如吧响应时间的毫秒转换成秒，然后跟阈值比较，大于阈值的可以直接输出个Fail之类的，自由发挥好了。



附上abs工具的用法：

	Usage: bin\abs.exe [options] [http[s]://]hostname[:port]/path 

	Options are:

	-n requests Number of requests to perform

	-c concurrency Number of multiple requests to make

	-t timelimit Seconds to max. wait for responses

	-b windowsize Size of TCP send/receive buffer, in bytes

	-p postfile File containing data to POST. Remember also to set -T

	-u putfile File containing data to PUT. Remember also to set -T

	-T content-type Content-type header for POSTing, eg.

	'application/x-www-form-urlencoded'

	Default is 'text/plain'

	-v verbosity How much troubleshooting info to print

	-w Print out results in HTML tables

	-i Use HEAD instead of GET

	-x attributes String to insert as table attributes

	-y attributes String to insert as tr attributes

	-z attributes String to insert as td or th attributes

	-C attribute Add cookie, eg. 'Apache=1234. (repeatable)

	-H attribute Add Arbitrary header line, eg. 'Accept-Encoding: gzip'

	Inserted after all normal header lines. (repeatable)

	-A attribute Add Basic WWW Authentication, the attributes

	are a colon separated username and password.

	-P attribute Add Basic Proxy Authentication, the attributes

	are a colon separated username and password.

	-X proxy:port Proxyserver and port number to use

	-V Print version number and exit

	-k Use HTTP KeepAlive feature

	-d Do not show percentiles served table.

	-S Do not show confidence estimators and warnings.

	-g filename Output collected data to gnuplot format file.

	-e filename Output CSV file with percentages served

	-r Don't exit on socket receive errors.

	-h Display usage information (this message)

	-Z ciphersuite Specify SSL/TLS cipher suite (See openssl ciphers)

	-f protocol Specify SSL/TLS protocol

	(SSL2, SSL3, TLS1 or ALL)
