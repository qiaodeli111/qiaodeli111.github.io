---
layout: post
title: "Websphere自动化管理方法集和脚本示例"
description: ""
category: "Middleware"
---
{% include JB/setup %}


作为IBM Websphere产品的运维工程师，我们都知道Websphere提供了两种进行管理的工具：基于WEB界面的图形化工具，还有wsadmin工具。

在做日常维护做一些小改动的时候，图形工具当然是最佳选择，但是在创建一个全新的环境或者在做迁移的时候，或者有一些固定的批量动作需要定时执行的时候，图形工具并不是最快最省力的方式，这时就需要用到wsadmin工具了。

<!-- more -->

wsadmin内置五大对象，每个对象负责特定的一些功能，我们可以通过JACL语法和Jython语法来调用这些内置对象。当然很明显Jython要比JACL强大的多，而且对于熟悉高级编程语言的人来说，Jython也更简单（跟Python语法一致），所以在编写自动化脚本的时候，基本的思路就是：Jython负责逻辑部分，调用wsadmin的内置对象实现特定的功能。


要说从头开始编写一套可以直接使用的脚本其实也是挺复杂的，正好我从IBM网站上找到了一个Websphere方法集，叫wasadminlib.py，既然有现成的那就用起来吧。本文主要目的是分享这个文件，并且介绍一下基本的使用方法。

## 下载

下载地址：[http://dellyqiao.qiniudn.com/2015/02/03/wsadminlib.py](http://dellyqiao.qiniudn.com/2015/02/03/wsadminlib.py)



## 如何使用

### 1. 创建一个个人的py文件，用来放置自己的逻辑代码。并且把这个文件跟wasadminlib.py放置在同一个文件夹下

这里我们创建一个createJAAS.py，用来创建JAAS认证别名。


### 2. 在createJAAS.py中引入wasadminlib.py

	execfile('wsadminlib.py')


**注意！这里要用execfile，不要用import**


### 3. 编写自己的逻辑代码

这里我把自己的创建JAAS的逻辑代码贴出来：


	execfile('wsadminlib.py')

	//我们要从JAAS.txt中读取需要创建的认证别名，用户名和密码。
	f = open("JAAS.txt")
	line = f.readline()
	while line:
	    print line
	    attrs = line.split(',')
	    //这里的createJAAS就是wasadminlib.py提供的方法
	    createJAAS(attrs[0], attrs[1], attrs[2])
	    line = f.readline()
	f.close()

	AdminConfig.save()


接下来也要创建一个JAAS.txt文件，内容如下：

	testing1, user1, testing1
	testing2, user2, testing2
	testing3, user3, testing3
	testing4, user4, testing4
	testing5, user5, testing5
	testing6, user6, testing6


这个文件里的三列分别代表：别名，用户名，密码。

**注意在编写JAAS.txt文件的时候，理论上是不应该有空格的，但是在我的测试环境Websphere8.0中，实际上是可以自动忽略前后的空格，所以加上也无所谓，观感上也会稍微好一点。如果不放心可以不加空格。**

### 4. 测试和调用

//跳转到脚本所在的位置
	cd C:\WASScript\

	//如果没有设置安全性，那可以不用添加 -username 和 -password 参数
	c:\IBM\WebSphere\AppServer\profiles\AppSrv01\bin\wsadmin.bat -lang jython -f createJAAS.py -username xxxx -password xxxx
	

测试的时候需要注意：

1. 可以先注释掉`AdminConfig.save()`，这行代码用来保存之前所有的配置，所以在确定语法以及一切都没问题之前，先不要让这句代码生效。
2. 如果想要测试wasadminlib.py的功能，可以直接用wsadmin工具测试。方法如下：
	
		//以Jython语法打开wsadmin工具
		c:\IBM\WebSphere\AppServer\profiles\AppSrv01\bin\wsadmin.bat -lang jython
		//引入wsadminlib.py
		execfile('wsadminlib.py')
		//现在就可以直接在这个交互的命令行里使用wsadminlib.py里面的方法了
	

## 脚本示例

上面的例子中我写了一个创建JAAS的方法，下面例举了创建DataSourceProvider和DataSource的方法：

### 创建DSProvider：

	execfile('wsadminlib.py')

	f = open("DBProvider.txt")
	line = f.readline()


	while line:
		attrs = line.split(',')
		print attrs
		
		//createJdbcProvider方法里的第一个参数server不是server的名字，而是Websphere为你的server生成的唯一的ID，如果想要把provider创建在Cell或Node下也一样，要找出对应的唯一ID。这个ID可以用wsadminlib.py提供的getXXXID()来找到，也可以从对应的scope的resources.xml文件里找到。
		serverID = getServerId(attrs[0],attrs[1])
		providerID = createJdbcProvider(serverID, attrs[2], attrs[3], attrs[4], attrs[5], attrs[6])
	f.close()

	AdminConfig.save()

相关的DBProvider.txt如下：

	//可以根据需求修改脚本和这个文件。比如如果你的provider是要创建到Node这个scope之下的，第二个参数“server1”就没用了，在脚本中也需要调用getNodeID(attrs[0])来获取到“dellyserverNode01”的唯一ID。
	dellyserverNode01,server1,mssql1,C:/drivers/mysql-driver.jar,C:/drivers/mysql-driver.jar,com.mysql.jdbc.jdbc2.optional.MysqlConnectionPoolDataSource,MySQL by Script,None


### 创建DataSource：

wsadminlib.py提供了两种创建DataSource的方法，这个是简单通用的那种，仅仅是创建一个基本的DataSource，具体的DB name，DB server name，等等，都需要手动去创建。另一个方法是createDataSource_ext，需要提供很多的参数，有兴趣的自己搞一下吧。

	execfile('wsadminlib.py')


	f = open("DataSource.txt")
	line = f.readline()

	while line:
	    attrs = line.split(',')
	    print attrs
	    
	    //注意：wsadminlib.py提供了一些获取某些ID（比如server，cell，node）的方法，但是没有提供获取JDBCProvider的唯一ID的方法，像这种需求就可以用getObjectByXXX来获取，除了byName还可以ByNodeName，等等。具体请查看wsadminlib.py本身。
	    providerID = getObjectByName(attrs[0])

	    print createDataSource(providerID, attrs[1], attrs[2], attrs[3], attrs[4], attrs[5], attrs[6])
	    line = f.readline()

	f.close()

	AdminConfig.save()


相关的DataSource.txt：



	mysql_driver,user,userstable,jdbc/user,10,testing1,com.ibm.websphere.rsadapter.GenericDataStoreHelper



----

所有脚本可以从这里下载：

[http://dellyqiao.qiniudn.com/2015/02/03/WASScript.zip](http://dellyqiao.qiniudn.com/2015/02/03/WASScript.zip)