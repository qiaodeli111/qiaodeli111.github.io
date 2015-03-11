---
layout: post
title: "用Shell生成Zabbix监控的数字报表"
description: ""
category: "Server"
---
{% include JB/setup %}

我们都知道Zabbix是一个非常强大的监控工具，我们公司也在用Zabbix监控所有网站的状态。

最近有一个需求，就是我们需要生成一份报告，报告里要包含前一天一整天的时间节点和对应的响应时间，这样的报告用来统计网站的可用性达到了什么水准。比如一个Excel格式的报表就ok。

Zabbix本身自带报告功能，可是都是图像的，并没有数据格式的。虽然没有提供数据格式的报表，但是Zabbix提供了一套API，可以根据需求获取对应的数据（JSON格式），于是我就写了个脚本，用来把每个监控的网址的监控数据输出到一个CSV文件中。（后续可以用Excel做一个宏去把所有收集到的CSV文件集合成一个XLS文件，这个Excel文件可以下载到：）

<!-- more -->

[README.xls](http://dellyqiao.qiniudn.com/2015/03/11README.xls)  


PS：服务器上面没有任何高级编程语言的编译工具或者运行环境，所以一切数据都用shell脚本去处理。用shell处理Json，蛋疼的不是一点点。。。。。。


**本脚本可以收集前一天所有监控的URL的响应速度数据，每个URL对应一个单独的CSV文件。收集完之后会把所有的CSV和README.xls打包发给预订的邮箱。当你收到这个zip包之后，解压，打开README.xls，点击里面的大按钮就可以生成一个集成的报表了，集成的报表中，每个URL对应一个excel标签，方便查看。**


脚本如下：

	#!/bin/bash  
	  
	##获取api的授权码，后面获取需要的信息的时候要用到。  
	  
	##先根据官方方法获取授权，之后会有一段输出信息，其中包含授权码，用awk处理一下就能得到那串码了。  
	authjson=`curl -l -X POST -H 'content-type: application/json' -d '{"jsonrpc":"2.0","method":"user.authenticate","params":{"user":"Admin","password":"zabbix"},"id":1}' http://127.0.0.1/zabbix/api_jsonrpc.php`  
	authstr=`echo $authjson | awk -F "[,:\"]" '{print $11}'`  
	echo $authstr  
	  
	##报告的初始时间和结束时间（前一天的0点到24点）  
	from=`date "+%Y-%m-%d 00:00:00" -d"-1day"`  
	echo $from  
	now=`date "+%Y-%m-%d 00:00:00"`  
	  
	##转换为Linux时间格式，Zabbix只支持这种格式。  
	from=`date -d "$from" '+%s'`  
	now=`date -d "$now" '+%s'`  
	  
	##每隔监控的网址在数据库中都有对应的一条记录，现在直接从数据库中获取所有的网址对应的ID。  
	  
	##getID.sql的内容如下：  
	  
	##select items.itemid from items join hosts on (items.hostid=hosts.hostid) where items.description like '%response time%' and hosts.host like '%WebSite%' and items.status=0; (sql文件可以根据实际情况编写，只要能得到所有受监控网址的ID就好。)  
	  
	  
	mysql --user=root --password=zabbix zabbix < getMalaysiaID.sql > /etc/scripts/outputMYID.txt  
	  
	  
	##mysql命令获取到的ID是有一个表头的，去掉。  
	sed '1d' /etc/scripts/outputMYID.txt > /etc/scripts/outputMYID_Daily.txt  
	  
	##清理以前的临时文件  
	rm -rf /etc/scripts/dailyreports/tmpjson/*.txt  
	  
	##获取每一个ID对应的网址的历史监控记录，Json格式，用时间段来做限定。  
	for i in `cat /etc/scripts/outputMYID_Daily.txt`  
	do  
	jsonstr="{\"jsonrpc\": \"2.0\",\"method\":\"history.get\",\"params\":{\"history\":0,\"itemids\":[\"$i\"],\"time_from\":\"$from\",\"time_till\":\"$now\",\"output\":\"extend\"},\"auth\": \"$authstr\",\"id\": 1}"  
	gethistory="curl -l -X POST -H 'Content-Type: application/json' -d '"$jsonstr"' http://127.0.0.1/zabbix/api_jsonrpc.php"  
	echo $gethistory > /etc/scripts/tmp.sh  
	chmod a+x /etc/scripts/tmp.sh  
	/etc/scripts/tmp.sh > /etc/scripts/dailyreports/tmpjson/$i.txt  
	done  
	  
	##新建一个文件夹!- -  
	now=`date "+%Y%m%d"`  
	mkdir /etc/scripts/dailyreports/dailyreports$now  
	  
	##处理获取到的Json数据了  
	for i in `ls /etc/scripts/dailyreports/tmpjson/`  
	do  
	rm /etc/scripts/dailyreports/tmp/*  
	  
	##获取所有的响应时间的记录  
	  
	cat /etc/scripts/dailyreports/tmpjson/$i | sed -e 's/[{}]/''/g' | awk -v k="text" '{n=split($0,a,","); for (i=1; i<=n; i++) print a[i]}' | grep value | grep -o '[0-9]\\.[0-9]*' > /etc/scripts/dailyreports/tmp/values  
	  
	##获取所有的时间节点的记录  
	cat /etc/scripts/dailyreports/tmpjson/$i | sed -e 's/[{}]/''/g' | awk -v k="text" '{n=split($0,a,","); for (i=1; i<=n; i++) print a[i]}' | grep clock | grep -o '[0-9]*' > /etc/scripts/dailyreports/tmp/clocks_org  
	  
	##获取到的时间节点要处理成人看的格式。  
	for c in `cat /etc/scripts/dailyreports/tmp/clocks_org`  
	do  
	date -d@$c '+%Y-%m-%d %H:%M:%S' >> /etc/scripts/dailyreports/tmp/clocks_new  
	done  
	  
	##设置新文件的文件名  
	itemidstr=`echo $i | awk -F [.] '{print $1}'`  
	itemname=`mysql --user=root --password=zabbix zabbix -e "select key_ from items where itemid=$itemidstr;" | awk -F [\\[,] '{print $2}'`  
	  
	##创建一个报告文件，并且加入表头  
	echo "Monitored URL:,$itemname," > /etc/scripts/dailyreports/dailyreports$now/$itemidstr.csv  
	echo "Clock,Response Time(s)," >> /etc/scripts/dailyreports/dailyreports$now/$itemidstr.csv  
	  
	##把时间节点的记录跟响应时间的记录合成一个文件，并且追加到刚刚的报告文件里。  
	paste -d "," /etc/scripts/dailyreports/tmp/clocks_new /etc/scripts/dailyreports/tmp/values /dev/null >> /etc/scripts/dailyreports/dailyreports$now/$itemidstr.csv  
	  
	done  
	  
	##清理临时文件  
	rm -rf /etc/scripts/dailyreports/tmpjson/*.txt  
	  
	##把所有获得的报告都收集起来，然后把写好宏的Excel文件也收集到一起，打包发给需要的人。  
	  
	##到时候在excel文件里会有一堆说明，点击哪儿哪儿的按钮就可以得到一份合适的Excel的报告啦。  
	mkdir /etc/scripts/dailyreports/dailyreports$now/csv  
	mv /etc/scripts/dailyreports/dailyreports$now/*.csv /etc/scripts/dailyreports/dailyreports$now/csv/  
	cp /etc/scripts/dailyreports/README.xls /etc/scripts/dailyreports/dailyreports$now/  
	cd /etc/scripts/dailyreports/  
	zip -r dailyreports$now.zip dailyreports$now/  
	  
	cd /root/sendEmail-v1.51  
	  
	./sendEmail –f sender@sender.com -t receiver@receiver.com -u KLDC DMZ1 Daily SLA Report for WebHosting -m KLDC DMZ1 Daily SLA Report For WebHosting -s 192.168.169.23:25 -a /etc/scripts/dailyreports/dailyreports$now.zip  
	  
	  
	rm -rf /etc/scripts/dailyreports/dailyreports*
