---
layout: post
title: "基于IP地址和基于hostname的两种批量配置Apache虚拟主机的方法"
description: ""
category: "Middleware"
---
{% include JB/setup %}

本文介绍如何从头开始为Apache配置多个虚拟主机。

Apache可以通过三种方式实现在同一台物理机上配置多个虚拟主机：

1. 基于不同的端口号
2. 基于不同的IP地址
3. 基于不同的hostname

本文介绍2和3。

注意：本文环境是Suse Linux，安装的Apache也是由Novell公司的Suse源里提供的Apache2.2。Apache的默认配置、文件的路径之类的可能会与其它版本的Apache有所不同，所以此文仅供参考。但是配置方法适用于所有2.2的Apache。

<!-- more -->

----

基于IP地址的虚拟主机设置。

假设当前物理机的IP地址是10.52.19.44，我们希望创建20个虚拟主机。这里用两个domain来做例子: www.domain1.com（10.52.19.60）, www.domain2.com（10.52.19.61）。步骤如下：

1. 为服务器配置虚拟IP。我们的主机上只有一块网卡，配置多个虚拟IP就可以用不同的IP来接收数据包。

	vim /etc/sysconfig/network/ifcfg-eth0， 然后更改如下：
		
		BOOTPROTO='static'
		BROADCAST=''
		ETHTOOL_OPTIONS=''
		IPADDR='10.52.19.44/24'
		GATEWAY='10.52.19.254'
		MTU=''
		NAME=''
		NETWORK=''
		REMOTE_IPADDR=''
		STARTMODE='auto'
		USERCONTROL='no'
		_nm_name='eth0'
		IPADDR_0='10.52.19.60'
		LABEL_0='1'
		NETMASK_0='255.255.255.0'
		IPADDR_2='10.52.19.61'
		LABEL_2='2'
		NETMASK_2='255.255.255.0'
		

	重启网络服务使设置生效：
	
		service network restart

2. 启动Apache的SSL服务。

	我们尽量遵守软件提供商希望的方法来配置我们的软件。对这个Apache的版本来说，所有SSL相关的设置都是要配置到`<IfDefine SSL>`标签中，所以只有当这个对应的参数配置过以后，Apache才会启动SSL，否则一切SSL的配置都不会生效。（当然也可以把所有SSL相关的配置都放到这个标签之外，但是这样的话如果我们想禁用SSL就会非常麻烦）

	vim /etc/sysconfig/apache2，修改如下：

		APACHE_SERVER_FLAGS="SSL"

3. 创建相关的文件夹：

	1) 为我们的Domain创建DocumentRoot文件夹，这里使用脚本test.sh：
	
		#!/bin/bash
		for i in `cat urls`
		do
		mkdir /Apps/$i
		echo "<h1>Index Page for $i</h1>" > $i/index.html
		done
	
	需要创建一个名为“urls”的文件并且放到与test.sh相同的路径下：
	
		www.domain1.com
		www.domain2.com

	修改脚本权限，运行：
	
		cd /Apps/
		chmod +x test.sh
		./test.sh
	
	2) 稍微修改脚本，创建相关的日志文件夹。 （比如把“／Apps“修改为”/Apps/logs“）

4. 我们使用自签名证书进行测试。创建自签名证书的方法:

		cd /etc/Apache2/SSL/
		openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout ssl.key -out ssl.crt

	命令执行后会有交互界面输入相关信息，关键的信息是”Common Name“，这儿要填写完整域名（比如www.domain1.com）
	
5. 配置Apache监听的IP和端口:

	vim /etc/apache2/listen.conf
	
		Listen 10.52.19.60:80
		Listen 10.52.19.61:80	
		
		<IfDefine SSL>
		    <IfDefine !NOSSL>
		        <IfModule mod_ssl.c>
		
		                Listen 10.52.19.60:443
		                Listen 10.52.19.61:443
		        </IfModule>
		    </IfDefine>
		</IfDefine>


6. 创建虚拟主机，这里使用脚本:

	1) 创建一个虚拟主机的模板，命名为”temp“放到/etc/apache2/vhost.d/。

		<VirtualHost 10.52.19.60:80>
		        ServerName www.domain.comwww.domain.com:80
		        DocumentRoot /Apps/www.domain.com
		        ErrorLog /Apps/logs/www.domain.com/error_log
		        LogLevel warn
		        CustomLog /Apps/logs/www.domain.com/access_log combined
		        RewriteEngine on
		
		</VirtualHost>
		
		<IfDefine SSL>
		<IfDefine !NOSSL>
		
		<VirtualHost 10.52.19.60:443>
		
		        DocumentRoot /Apps/www.domain.com
		        ServerName www.domain.com:443
		        ErrorLog /Apps/logs/www.domain.com/error_log
		        TransferLog /Apps/logs/www.domain.com/access_log
		
		        SSLEngine on
		        # 4 possible values: All, SSLv2, SSLv3, TLSv1. Allow TLS only:
		        SSLProtocol all -SSLv2 -SSLv3
		
		        #   "openssl ciphers -v" for a complete list of ciphers that are
		        #   available.
		        SSLCipherSuite ALL:!aNULL:!eNULL:!SSLv2:!LOW:!EXP:!MD5:@STRENGTH
		
		        SSLCertificateFile /etc/apache2/SSL/ssl.crt
		        #SSLCertificateFile /etc/apache2/ssl.crt/server-dsa.crt
		
		        SSLCertificateKeyFile /etc/apache2/SSL/ssl.key
		        #SSLCertificateKeyFile /etc/apache2/ssl.key/server-dsa.key
		
		        #SSLCertificateChainFile /etc/apache2/ssl.crt/ca.crt
		
		        #SSLCACertificatePath /etc/apache2/ssl.crt
		        #SSLCACertificateFile /etc/apache2/ssl.crt/ca-bundle.crt
		        #SSLCARevocationPath /etc/apache2/ssl.crl
		        #SSLCARevocationFile /etc/apache2/ssl.crl/ca-bundle.crl
		
		        #SSLVerifyClient require
		        #SSLVerifyDepth  10
		        <Files ~ "\.(cgi|shtml|phtml|php3?)$">
		            SSLOptions +StdEnvVars
		        </Files>
		        <Directory "/srv/www/cgi-bin">
		            SSLOptions +StdEnvVars
		        </Directory>
		        CustomLog /Apps/logs/www.domain.com/ssl_request_log   ssl_combined
		
		</VirtualHost>
		
		</IfDefine>
		</IfDefine>

	2) 在同一个文件夹下创建文件"urls"，内容如下:
		
		www.domain1.com 10.52.19.60
		www.domain2.com 10.52.19.61
		
	3) 建立一个脚本(test.sh)，内容如下:

		#!/bin/bash
		while read line
		do
				##提取出domain name
		        HOST=`echo $line | awk '{print $1}'`
		        ##提取出对应的IP
		        IP=`echo $line | awk '{print $2}'`
		        ##修改虚拟主机中的domain name和IP，然后建立对应的虚拟主机配置文件
		        sed -e 's/10.52.19.60/'$IP'/g' -e 's/www.domain.com/'$HOST'/g' temp > vhost-$HOST.conf
		        echo "Virtual Host for $HOST has been created!"
		
		done < urls

	4) 脚本加权限执行：
	
		cd /etc/apache/vhost.d/
		chmod +x test.sh
		./test.sh

7. 重启Apache服务：

		service apache2 restart
		
*如何测试虚拟主机是否生效：*

*在测试的PC上，打开浏览器输入对应的IP地址，看看会不会跳转到对应的index.html页面*

********

基于hostname的虚拟主机设置。

假设当前物理机的IP地址是10.52.19.44，我们希望创建20个虚拟主机。这里用两个domain来做例子。虚拟主机拥有不同的主机名，但是我们希望用同一个IP。 例如：www.domain1.com（10.52.19.44）, www.domain2.com（10.52.19.44），我们希望通过www.domain1.com访问Apache服务器的时候，用户访问到domain1的所有文件；通过www.domain2.com访问Apache服务器的时候，用户访问到domain2的所有文件。

为了可以方便地批量创建虚拟主机，我写了一个脚本实现所有功能。首先可以看看我们文件夹下面的所有文件列表：

	/tmp/configure # ls -R
	.:
	SSL  test.sh  urls  vhost-temp
	
	./SSL:
	www.domain1.com  www.domain2.com
	
	./SSL/www.domain1.com:
	ssl.crt  ssl.key
	
	./SSL/www.domain2.com:
	ssl.crt  ssl.key
	
test.sh内容如下:

	#!/bin/bash
	
	## Define variables
	today=`date +"%Y%m%d"`
	ipaddr=`ifconfig eth0 | grep "inet addr" | awk -F ":" '{print $2}' | awk '{print $1}'`
	
	## Make the required directories and copy required files
	mkdir /Apps/logs/
	mkdir /etc/apache2/backup_$today
	cp -r SSL/ /etc/apache2/
	cp httpd.conf /etc/apache2/
	
	for i in `cat urls`
	do
	        mkdir /Apps/$i
	        mkdir /Apps/logs/$i
	        echo "<h1>Index Page for $i</h1>" > /Apps/$i/index.html
	        sed -e 's/www.domain.com/'$i'/g' vhost-temp > /etc/apache2/vhosts.d/vhost-$i.conf
	        echo "Virtual Host for $i has been created!"
	done
	
	
	## Backup the modified files
	cd /etc/apache2
	cp httpd.conf backup_$today
	cp listen.conf backup_$today
	
	##Change listen.conf
	echo "NameVirtualHost $ipaddr:80" >> listen.conf
	echo "NameVirtualHost $ipaddr:443" >> listen.conf
	sed -i -e 's/Listen 80/Listen '$ipaddr':80/g'  -e 's/Listen 443/Listen '$ipaddr':443/g' listen.conf
	
	##Restart service
	service apache2 restart


urls:

	www.domain1.com
	www.domain2.com
	
vhost-temp:

	<VirtualHost 10.52.19.44:80>
	        ServerName www.domain.com
	        DocumentRoot /Apps/www.domain.com
	        ErrorLog /Apps/logs/www.domain.com/error_log
	        LogLevel warn
	        CustomLog /Apps/logs/www.domain.com/access_log combined
	        RewriteEngine On
	        RewriteCond %{REQUEST_METHOD} !^(GET|HEAD|POST)$
	        RewriteRule .* - [F]
	        #RewriteCond %{SERVER_PORT} !^443$
	        #RewriteRule ^/(.*)$ https://%{SERVER_NAME}/$1 [L,R]
	        RewriteRule ^/$ http://%{SERVER_NAME}/index.html
	
	</VirtualHost>
	
	
	<IfDefine SSL>
	<IfDefine !NOSSL>
	
	<VirtualHost 10.52.19.44:443>
	
	        ServerName www.domain.com
	        DocumentRoot /Apps/www.domain.com
	        ErrorLog /Apps/logs/www.domain.com/error_log
	        TransferLog /Apps/logs/www.domain.com/access_log
	
	        RewriteEngine On
	        RewriteCond %{REQUEST_METHOD} !^(GET|HEAD|POST)$
	        RewriteRule .* - [F]
	        #RewriteCond %{SERVER_PORT} !^443$
	        #RewriteRule ^/(.*)$ https://%{SERVER_NAME}/$1 [L,R]
	        RewriteRule ^/$ https://%{SERVER_NAME}/index.html
	
	
	        SSLEngine on
	        # 4 possible values: All, SSLv2, SSLv3, TLSv1. Allow TLS only:
	        SSLProtocol all -SSLv2 -SSLv3
	
	        #   "openssl ciphers -v" for a complete list of ciphers that are
	        #   available.
	        SSLCipherSuite ALL:!aNULL:!eNULL:!SSLv2:!LOW:!EXP:!MD5:@STRENGTH
	
	        SSLCertificateFile /etc/apache2/SSL/www.domain.com/ssl.crt
	        #SSLCertificateFile /etc/apache2/ssl.crt/server-dsa.crt
	
	        SSLCertificateKeyFile /etc/apache2/SSL/www.domain.com/ssl.key
	        #SSLCertificateKeyFile /etc/apache2/ssl.key/server-dsa.key
	
	        #SSLCertificateChainFile /etc/apache2/ssl.crt/ca.crt
	
	        #SSLCACertificatePath /etc/apache2/ssl.crt
	        #SSLCACertificateFile /etc/apache2/ssl.crt/ca-bundle.crt
	        #SSLCARevocationPath /etc/apache2/ssl.crl
	        #SSLCARevocationFile /etc/apache2/ssl.crl/ca-bundle.crl
	
	        #SSLVerifyClient require
	        #SSLVerifyDepth  10
	
	        <Files ~ "\.(cgi|shtml|phtml|php3?)$">
	            SSLOptions +StdEnvVars
	        </Files>
	        <Directory "/srv/www/cgi-bin">
	            SSLOptions +StdEnvVars
	        </Directory>
	        CustomLog /Apps/logs/www.domain.com/ssl_request_log   ssl_combined
	
	</VirtualHost>
	
	</IfDefine>
	</IfDefine>


加权限执行test.sh即可。

*如何测试虚拟主机是否工作正常：*

*在测试用的PC上添加如下内容到hosts文件（C:\Windows\System32\drivers\etc\hosts for Windows, /etc/hosts for Lunix/Max)*

	10.52.19.44	www.domain1.com
	10.52.19.44	www.domain2.com
	
*随后打开浏览器，访问对应的域名，查看是否访问到对应域名下的文件*


********

基于端口的虚拟主机设置与基于IP的很类似，只要修改一下监听的端口和虚拟主机中的相关配置即可。