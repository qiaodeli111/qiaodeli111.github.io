---
layout: post
title: "Hadoop Cluster集群配置心得（低配置集群+自动同步配置）"
description: ""
category: "Hadoop"
---
{% include JB/setup %}


本文为本人原创，首发到炼数成金 <http://f.dataguru.cn/thread-138720-1-1.html>。



情况是这样的，我没有一个非常强劲的电脑来搞出一个性能非常NB的服务器集群，相信很多人也跟我差不多，所以现在把我的低配置集群经验拿出来写一下。

  


我的配备：1）五六年前的赛扬单核处理器2G内存笔记本 2）公司给配的ThinkpadT420，i5双核处理器4G内存（可用内存只有3.4G，是因为装的是32位系统的缘故吧。。。）

<!-- more -->


就算是用公司配置的电脑，做出来三台1G内存的虚拟机也显然是不现实的。企业笔记本运行的软件多啊，什么都不做空余内存也才不到3G。所以呢，我的想法就是：

  


用我自己的笔记本（简称PC1）做Master节点，用来跑Jobtracker,Namenode 和SecondaryNamenode；用公司的笔记本跑两个虚拟机（简称VM1和VM2），用来做Slave节点，跑Tasktracker和Datanode。这么做的话，就需要让PC1，VM1和VM2处于同一个网段里，保证他们之间可以互相连通。

  


网络环境：我的两台电脑都是通过一个无线路由上网。

  


### **1. 构建跟外部的电脑同一网段的虚拟机配置过程：**

  


准备工作：构建一个集群，首先前提条件是每台服务器都要有一个固定的IP地址，然后才可能进行后续的操作。所以呢，先把我的两台笔记本电脑全部设置成固定IP（注意，如果像我一样使用无线路由上网，那就要把无线网卡的IP设置成固定IP）。用来做Master节点的PC1:192.168.33.150，用来跑虚拟机的宿主笔记本：192.168.33.157。目标：VM1和VM2的IP地址分别设置成192.168.33.151和152。

  


步骤：

  


1）新建VM1虚拟机。

  


2）打开VM1的网卡设置界面，连接方式选Bridge。（桥接）

  


关于桥接的具体信息，可以百度一下。我们需要知道的，就是用桥接的方式，可以让虚拟机通过本机的网关来上网，所以就可以跟本机处于同一个网段，互相之间可以进行通信。

  


3）我用的是VMware Workstation8，就以他为例：菜单Edit——VirtualNetwork Editor。

  


4）选择WMnet0（这个是VMware安装时默认预留出来为桥接这种连接方式做的一个配置文件），然后点击下面的bridgedto，就是要连接到哪个网卡上，要通过哪个网卡来上网。如果选择“自动“不可以正常上网的话，就手动选，选择连接到网络的那块网卡就可以了。（比如我的是无线网卡连路由器上网的，所以这里就选我自己的无线网卡就好）

  


5）然后，点击OK。进入系统内部，设置IP地址为固定IP：192.168.33.151。我使用图形界面设置的。具体不同的Linux系统请查阅相关的网络配置文档。
  


6）现在就可以互相ping一下了。如果可以ping通，那就说明互相连上了。

  


如果不能，那就ifconfig查看一下各自的IP是否设置成功了，是否需要重新连接，之类的。

  


按照这个步骤来，应该是没什么问题的。。。如果有哪些地方不明白或者设置出问题的，可以告诉我，互相交流看看。

  


### **2. SSH**设置：

  


虚拟机设置好之后就可以设置SSH了。当然在这之前要先建立好账户，三台服务器都建立一个相同的账户。

  


Ubuntu默认是没有SSH server的（CentOS默认可以选择安装上），所以按照老师视频里设置好SSHKey之后是不可能互相SSH到的。（可以通过各种命令验证，比如`telnet localhost 22`来验证到底本机上的22端口有没有程序在监听，显示连接失败的话那当然就是SSH Server没有设置咯。或者通过`netstat –ano | grep ssh`之类的命令来查看。）

  


关于SSH的安装，貌似现在OpenSSH是属于Linux系统上实际的SSH实现标准了，Ubuntu也可以很方便的安装OpenSSH。命令如下：

	sudo apt-get installopenssh-server

  


一行命令就够了。装完之后telnet试试先，可以验证是否安装成功。

  


然后再按照老师视频里面的办法，生成公私密钥，把公钥全部聚在一起传播到每台服务器上。就可以实现无密码登陆了。

  


**注意！在authorized_key****放到指定位置之后，千万要手动SSH****到所有的节点上一次！**

  


比如像我的，要从PC1输入命令：ssh <VM1 IP>和ssh <VM2IP>来手动SSH到另外的所有节点上一次，看到是否要信任目标主机的提示后，输入yes，然后这才算是真的设置完成了。这是一个必要的操作，只有这样才可以把其他所有server的公钥都添加到本机的信任列表里，才能真的实现无任何提示的免密码登入。

  


否则，在启动其他节点上的守护进程时会显示无法连接到对应的节点，无法启动之类的提示。

  


这样的话，我的低配置集群环境就大概准备好了。

  


### **3. 接下来就是配置Hadoop了。**

  


关于Hadoop的配置，我的想法：这么大个集群，肯定得有个同步的机制吧，不然每次改一下配置文件就需要在所有的节点上改一次，那得是多大的工作量啊。。。。。。于是，看了一下权威指南教材269页开始的配置，确实是有这个配置的。此外还有一些配置是老师没有讲到的，也一并以我自己的理解和实践为例，写出来吧。


主要的配置文件其实就是hadoop-env.sh和core-site.xml，hdfs-site.xml和mapred-site.xml。

  


#### **1. hadoop-env.sh：**

首先，配置JAVA_HOME。这个不用说了，从oracle下载最新的gz包直接解压，设置好路径就好。

然后这些配置我觉得非常有用：

	export HADOOP_HEAPSIZE=400
	
这个是用来设置分配给守护进程的内存空间的。守护进程其实也全部都是Java进程，可以通过这个来设置JVM的Heapsize。我这里设置的是400MB。默认的是1000MB。


正好也说一下我的设计。因为我的是低配置的，所以如果全部都按照默认的设置来，比如我自己的那台2G笔记本上跑Jobtracker，Namenode和SecoundaryNamenode，每个进程分配1000M，那如果真的是有大量的数据进来了，分配的这个内存空间就会被充满，到时候我的电脑绝对崩溃。

  


所以，我的设计方案如下：

- *我的老笔记本（2G内存）：Jobtracker：400M，Namenode：800M，SecondaryNamenode：800M。（照我的理解，Namenode对内存和IO进行集中的管理，所以是需要大量的空间来运作的）*
- *虚拟机VM1和2（每台1.2G内存）：Tasktracker：400M，Datanode：400M，然后Tasktracker开启的Map任务和Reduce任务子进程各1个，每个分配200M内存。*


个人感觉这样至少电脑不会爆掉吧。。。。。。等到以后真的开始进行数据处理的时候再看看怎么个情况吧，现在也只是根据自己粗浅的认识来这么配置的。

	export HADOOP_NAMENODE_OPTS="-Xmx800m -Dcom.sun.management.jmxremote $HADOOP_NAMENODE_OPTS"

	export HADOOP_SECONDARYNAMENODE_OPTS="-Xmx800m -Dcom.sun.management.jmxremote $HADOOP_SECONDARYNAMENODE_OPTS"

	##这两个配置就是所说的Namenode和SecondaryNamenode分配800M的Heapsize。根据验证，这样设置是没问题的。

	export HADOOP_MASTER=dellypc-master:/home/$USER/hadoop-0.20.2

	host:path where hadoop code should be rsync'd from. Unset by default.
	#hadoop的code从哪里进行同步，默认为不同步。只要设置好了这个，就可以在每次更改配置文件的时候只更改一份配置文件，启动的时候启动信息里可以看到，所有的节点都会从设置的位置同步配置文件。

比如，我的Master节点hostname是dellypc-master，hadoop的所有配置文件都放在了/home/delly/hadoop-0.20.2这里（$USER会被识别成当前的用户名，也就是delly），那么每次启动hadoop守护进程的时候，hadoop都会自动先从dellypc-master的/home/delly/hadoop-0.20.2这个位置把所有的配置文件全部同步下来，然后才会进行启动的操作。

**值得注意的是，这个配置文件的保存地不一定非得是hadoop集群中的某一台机器，根据权威指南文档说的，可以是外部的一台server。**这样其实还挺好的，真的用起来的时候可以搞一台专门用来存放配置的机器，服务器集群肯定要通过防火墙隔离到DMZ网络中，然后开通这台配置机到集群机器的RPC文件同步端口，然后每次需要改配置就直接通过这台集群外的机器更改，并且启动集群守护程序的时候同步到所有节点，非常的安全和好用啊。

继续配置：

	export HADOOP_SLAVE_SLEEP=0.1
	
	##Seconds to sleep between slave commands. Unset by default. This can be useful in large clusters, where, e.g., slave rsyncs can otherwise arrive faster than the master can service them.
	##不知道具体原理是怎么回事，但是这个配置看起来好像是在启动的时候，主节点会抽空闲时间休息0.1秒钟，避免请求同时到达主节点导致负载过高宕掉。

hadoop-env.sh的配置就写到这里了，还有其他log位置，pid位置之类的，就自行设置一下就好。部分设置都是似懂非懂的感觉好像有用就设置了，也许以后会理解更深一点可以知道怎样设置地更正确。


#### 2. core-site.xml：

除了视频教程里说的设置hdfs的位置和端口之外（相信`fs.default.name`也是指定了namenode运行在哪台server上的吧。。），还配置了下面的东西：

	<property>
	<name>io.file.buffer.size</name>
	<value>131072</value>
	<!-- 设置缓冲区辅助IO操作，默认4K，这里设置成128KB -->
	<final>true</final>
	</property>

  


#### 3. hdfs-site.xml：

也是除了视频中的设置，还设置了：

	<property>
	<name>dfs.name.dir</name>
	<value>/home/delly/hadoop-0.20.2/name,/media/backup/hadoop-backup/namedata</value>
	<final>true</final>
	</property>

Namenode存放永久性元数据的目录，这里设置了两个目录，万一其中一个目录中数据出问题了，那么另一个作为冗余数据可以进行恢复


	<property>
	<name>fs.checkpoint.dir</name>
	<value>/home/delly/hadoop-0.20.2/namesecondary,/media/backup/hadoop-backup/namesecondarydata</value>
	<final>true</final>
	</property>

同上，只不过是secondaryNamenode存放的位置


	<property>

	<name>dfs.data.dir</name>

	<value>/home/delly/hadoop-0.20.2/data</value>

	</property>

	（datanode存放数据块的位置，默认是tmp文件夹，不安全吧。。。）

	<property>

	<name>dfs.replication</name>

	<value>2</value>

	</property>

（数据复制的份数，不太理解到底是什么数据的复制，有懂的同学求指教啊）

	<property>

	<name>dfs.block.size</name>

	<value>134217728</value>

	<!-- 128MB -->

	</property>

（HDFS块大小，默认64M，这里设置成128M，减小namenode内存压力）

  


#### 4. 然后mapred-site.xml文件：

主要是额外配置了如下的东西，指定map任务和reduce任务子进程最多有多少个，以及他们的内存（我分配了200M）

	<property>

	<name>mapred.tasktracker.map.tasks.maximum</name>

	<value>1</value>

	<final>true</final>

	</property>

	  


	<property>

	<name>mapred.tasktracker.reduce.tasks.maximum</name>

	<value>1</value>

	<final>true</final>

	</property>

	  


	<property>

	<name>mapred.child.java.opts</name>

	<value>-Xmx200m</value>

	</property>

	  


主要配置的东西就写到这里吧。

  


我现在有一个巨大的疑问，hadoop是如何控制哪个进程在哪个节点上运行的？

  


在配置文件里，我只看到了hdfs的主机（这个应该就是会指定namenode所在的主机吧？），然后还有jobtracker所在的主机，但是，SecondaryNamenode和Tasktracker，datanode到底要在哪个节点上执行，配置文件里完全没有配置，但是SecoundaryNamenode确实在我预想的主节点上跑起来了，datanode和tasktracker确实在两个slave节点上跑起来了，这是为什么？

  


有人能解答疑问吗？非常感谢！

  


经过测试和查看资料，这个疑问已解决： <http://f.dataguru.cn/thread-138720-1-1.html>
