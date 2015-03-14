---
layout: post
title: "Zabbix   Update from 1.8.3 to 1.8.12"
description: ""
category: "Monitoring"
---
{% include JB/setup %}

This document is to introduce how to upgrade the Zabbix from 1.8.3 to 1.8.12 to suit OpenSuse 11.4.

*Note: As this is not a full new configuration, so this document is only for Zabbix version update from 1.8.3 to 1.8.12, cannot be treated as a full installation guide.*

As this is a little update during 1.8.x, so no need to touch the DB. Steps attached here, please follow the steps strictly, all the command are provided here.

<!-- more -->

### Reference:

You may find more information from below links:

Install zabbix from Source: [https://www.zabbix.com/documentation/1.8/manual/installation/installation_from_source](https://www.zabbix.com/documentation/1.8/manual/installation/installation_from_source)

Upgrade proceduler: (section 3.6) [https://www.zabbix.com/documentation/1.8/manual/about/installation_and_upgrade](https://www.zabbix.com/documentation/1.8/manual/about/installation_and_upgrade)

Dependency packages: (Chinese) [http://os.51cto.com/art/201104/252989.htm](http://os.51cto.com/art/201104/252989.htm)


### Download:

You may get all the required packages from here: [Download](http://dellyqiao.qiniudn.com/2015/03/13/Zabbix_Update_packages.zip)


### Steps:

1. Remove the old version Zabbix 1.8.3.   

	You may run: `rpm -qa | grep zabbix` to check all the installed zabbix packages. Please confirm all the installed packages have been removed before perform the new installation.  

	Before remove:  

	![](http://dellyqiao.qiniudn.com/2015/03/13/1.png)
	   
	After:  

	![](http://dellyqiao.qiniudn.com/2015/03/13/2.png)


2. Install the dependency packages: 
  
		gcc  
		make  
		mysql-devel  
		curl-devel  
		net-snmp-devel 
		
	(Above packages could be find from the OpenSuse 11.4 OS iso file, and most of the Linux distrubution should have includes these tools already.)  
  
		iksemel-1.4.tar.gz 
		
	*install this package with below steps*:

	  1. Upload file iksemel-1.4.tar.gz to /tmp, then `cd /tmp`

	  2. `tar –zxf iksemel-1.4.tar.gz`

	  3. `cd iksemel-1.4`

	  4. `./configure`

	  5. `make`

	  6. `make install`


3. Upload the file zabbix-1.8.12.tar.gz to /tmp, then `cd /tmp`.

4. `tar –zxf zabbix-1.8.12.tar.gz`

5. `cd zabbix-1.8.12`

6. `./configure --enable-server --enable-proxy --enable-agent --with-mysql --with-net-snmp --with-jabber --with-libcurl`

7. `make install`

8. `ln -sf /usr/local/lib/libiksemel.so.3 /lib/`


	Basically, till now, the new version Zabbix have been installed, please run `zabbix_server –V` to verify the version of zabbix.

9. Copy the config file to folder: /etc/zabbix:  
	
		cd /tmp/zabbix-1.8.12  
		cp misc/conf/zabbix_server.conf /etc/zabbix/  
		cp misc/conf/zabbix_proxy.conf /etc/zabbix/  
		cp misc/conf/zabbix_agent.conf /etc/zabbix/  
		cp misc/conf/zabbix_agentd.conf /etc/zabbix/

10. `vi /etc/zabbix/zabbix_server.conf`  

	Find out below 2 lines, then modify the DB username and password: 
	
	(find function in VI: type `/ DBUser` in command mode)  
		
		DBUser=zabbix  
		DBPassword=zabbix

11. Upload the files: zabbix_server and zabbix_agentd to /etc/init.d/, then add execute permission:</div>

		chmod +x /etc/init.d/zabbix_server  
		chmod +x /etc/init.d/zabbix_agentd

12. Start and stop the zabbix_server and zabbix_agentd to verify the installation with below commands:  

		/etc/init.d/zabbix_server start  
		/etc/init.d/zabbix_server stop  
		ps –ef | grep zabbix  
		/etc/init.d/zabbix_agentd start  
		/etc/init.d/zabbix_agentd stop  
		ps –ef | grep zabbix  

	After confirmed there's no issue, start the zabbix_server and zabbix_agentd:
	
		/etc/init.d/zabbix_server start  
		/etc/init.d/zabbix_agentd start  		

----

Steer into the web interface configuration:

1. `cd /tmp/zabbix-1.8.12/frontends/php/`

2. `mkdir /srv/www/htdocs/zabbix`

3. `cp -a . /srv/www/htdocs/zabbix`

4. Now we don't need to work with the command line any more, instead, we need to open a Web browser to access the zabbix web interface to configure our web view.

	Open a browser, input the address: `http://<IP address here>/zabbix/queue.php`
	
	![](http://dellyqiao.qiniudn.com/2015/03/13/3.png)

	- If you are working locally on the Linux server, you may open the embeded Firefox (I guess all the popular Linux distribution have Firefox installed by default). 

	If you are working with Linux server remotely, you will have some choices:

	- Find out any PC which could access the remote server via port 80, then use PC browser to access the web page.
	- Configure remote X session from local PC, you may use Putty+xming (free), or xterm (not free), and then access the Firefox on the server from local PC directly.


5. Click `Next>>` Button. Then read and accept GPL v2 license.

	![](http://dellyqiao.qiniudn.com/2015/03/13/4.png)

6. Make sure that all software pre-requisites are met.

	![](http://dellyqiao.qiniudn.com/2015/03/13/5.png)

7. Configure database settings. Zabbix database must already be created.  
***Please input “zabbix” in “Name”, “User” and “Password” fields.***. 

	Then click `Test Connection` button, "Ok" means connection is fine.

	![](http://dellyqiao.qiniudn.com/2015/03/13/6.png)

8. Click Next. See summary of settings. Click next.

	![](http://dellyqiao.qiniudn.com/2015/03/13/7.png)
	
	![](http://dellyqiao.qiniudn.com/2015/03/13/8.png)

9. Download configuration file and place it under conf/.

	1. Click the “Save configuration file” button.

		![](http://dellyqiao.qiniudn.com/2015/03/13/9.png)

	2. After saved, upload the file zabbix.conf.php to /srv/www/htdocs/zabbix/conf/. 

		![](http://dellyqiao.qiniudn.com/2015/03/13/10.png)

10. Click `Retry` button, there should be "Ok" appears. Then finishing installation.

	![](http://dellyqiao.qiniudn.com/2015/03/13/11.png)


Till now, all the configuration is done, input the monitoring address on browser `http://IP Address/zabbix/` to work with the new environment.
