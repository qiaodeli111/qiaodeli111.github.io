---
layout: post
title: WebSphere应用服务器学习笔记
category: Middleware
date: 2015-03-09
---

WebSphere是应用程序服务器，是电子商务基础结构的核心。WebSphere服务器支持J2EE规范。

********

## Websphere的架构

Websphere采用三层架构来平衡工作负载，病最大化对资源的访问，从而提高了吞吐量和性能。

1. **表示层**：支持网络上的客户端计算机，向其展示信息，并且处理用户操作。

2. **应用程序逻辑层**：包含用于管理连接和处理数据的应用程序组建，包括servlet，EJB组建，applet等。

3. **资源层**：包含数据库和其他数据存储设备。该层中的服务位于安全网络中，客户端不能直接访问。


## Websphere的组件

WAS包含多个组件，每个组件都向平台提供特定的功能：

### EJB容器

安装在服务器上的EJB不直接与应用程序服务器交互。EJB容器就是服务器与EJB之间的接口。这是支持线程、事务处理和数据管理的服务器进程，处理进入和出自EJB的所有请求。

<!-- more -->

### WEB容器

WEB容器支持用于处理HTTP客户端请求的servlet和JSP文件，然后向其他应用程序组件提供数据，必要时还生成XML。

### 应用程序客户端容器

处理客户端一侧的Java程序，管理与应用服务器上承载的J2EE组件之间的交互，并在客户机上配置器执行环境。

### Applet容器

WEB服务器可以把生成WEB页时涉及的某些处理负荷转移到客户端，这是通过与普通HTML一起向其发送Java applet来实现的。这些applet是在客户端浏览器中运行的Java类，在客户端机器上安装的applet通过applet容器可以访问应用程序服务器上的EJB。

### 嵌入式HTTP服务器

WEB客户端通过HTTP服务器上的HTTP插件连接到应用程序服务器。嵌入式HTTP服务器是提供替代方案的Websphere内部组件，它使得客户端可以直接连接到应用程序服务器。

### 虚拟主机

在多台机器上运行Websphere的优点之一是可以隔离数据，使得只有同一台机器上安装的资源才能访问。你可以通过创建虚拟主机在一台机器上复制该功能，也就是说，让一台机器看起来仿佛是多台机器在运行。这样就可以限制对数据的访问，在虚拟机上运行具有自定义配置的应用程序。


## J2EE功能

Java 2 Enterprise Edition平台是在开发和部署企业应用程序中使用的一项标准。它也为J2EE应用程序提供运行库环境。

### J2EE标准定义的架构包含

- 开发多层应用程序的标准应用程序模型
- 承载应用程序的标准平台
- 保证达到J2EE标准的兼容性测试套件
- 提供J2EE平台运行定义的引用实现

### J2EE平台规范为J2EE应用程序定义运行库环境，包括：

- 应用程序组件
- 容器
- 资源管理器驱动程序

这些元素与一组也是由J2EE标准规定的标准设备进行通信。


### J2EE平台定义在应用程序开发和部署期间执行的一系列角色。理解这些角色可以更有效的开发和部署应用程序。

产品提供商 & 工具提供商 --》 产品

应用程序组件提供商 & 应用程序汇编器 --》 应用程序

部署者 & 系统管理员 --》 运行库

### 在J2EE架构中部署应用程序有多项优点：

- **简化的结构**：因为J2EE规范是以标准的Java组件和服务为基础的，所以企业应用程序只需编写一次，在任何地方都可以运行。
- **选择工具**：开发人员可以从一系列应用程序开发工具和现成的组件和解决方案中选择。
- **集成服务**：你可以把J2EE应用程序与当前多种系统集成到一起，包括Java数据库互联（JDBC）、Java消息服务（JMS）、Java接口定义语言（Java Interface Definition Language，Java IDL）和JavaMail及Java事务处理API。
- **缩放能力**：例如，你可以向上扩展J2EE架构来满足需求，把容器在多个系统上分布并使用数据库连接池。
- **安全性**：应用程序组件安全机制是统一而灵活的，你可以把它与集成安全系统集成到一起。


### J2EE编程模型有四种类型的应用程序组件，每种都在应用程序服务器中不同类型的容器内。

- EJB （在EJB容器中）
- servlet和JSP页面文件 （在WEB容器中）
- 应用程序客户端 （在应用程序客户端容器中）
- applet （在applet容器中）

每种类型的组件都有自己的容器。容器能够位组件提供运行库支持，为访问服务提供API，还能够处理安全机制、资源共享和其他问题。


### J2EE平台为实现组件之间的交互而提供了一组标准服务

其中包括HTTP和HTTPS、Java事务处理API和远程方法调用/Internet ORB间协议（Remote Method Invocation/Internet Inter-ORB Protocol, RMI/IIOP）。

### J2EE平台的一项重要特点是它打包应用程序进行部署的方式。

它把组件汇编到模块中，然后打包到应用程序。叫做部署描述符的XML文件控制每个模块和应用程序的部署


### 应用程序和EJB的新功能

WebSphere包含针对应用程序和EJB模块的新功能。EJB持久化管理器支持EJB2.0容器管理的持久化（container-managed persistence，CMP）方案，这与EJB1.1方案不同，这是对模块性、易维护性和性能的进一步完善。

EJB2.0的规范支持包括：

- 本地、远程和消息驱动的Bean
- 容器管理的关系和关联关系
- 便携式查找着查询语言
- 编程模式
- 抽象和具体实体bean
- 本地家庭和本地实体接口
- EJB查询语言

WebSphere5.0中提供的超过EJB2.0规范的高性能的持久化功能包括：

- 修改语义行为
- 实体bean的集成
- 乐观的并发控制
- 提前读取
- 密切的机制支持
- 后端访问支持
- SQL
- 数据高速缓冲

在WebSphere中完善CMP实体bean性能的新功能包括：

- bean数据高速缓存
- 长生存期高速缓冲
- 乐观的并发控制
- 提前读取

活动对话是WebSphere企业版的一项新功能，用户可以通过该对话把事务处理分组到个工作单位。你可以在各种属性和配置与活动对话之间建立关联。

CMP bean和bean管理的持久化（bean-managed persistence，BMP）bean可以通过共享数据存储连接在同一事务处理中访问数据。

CMP bean之间可以互相集成，或互为子集。在关系数据存储中，可以在单张表格或根叶布局中定义。应用程序服务器在部署时认可继承。



********


## 确定WebSphere平台基础结构

### Base版

在基本版本的运行库架构中有多个组件，与管理相关的组件包括：

- **节点**：WebSphere管理的服务器进程的逻辑安排，这些进程工作在公共配置和运行功能之外。
- **配置存储库**：配置库在XML文件中保存每个组件配置文档的副本。应用程序服务器的管理服务管理配置，并保证运行期间的连贯性。
- **虚拟主机**：通过虚拟主机可以把独立的一台主机当做多台主机使用。采用这项技术，一台物理机器可以支持一系列的唯一配置并管理的应用程序。虚拟主机不连接到特定节点上。是可以被创建但不能被启动和停止的组件。
- **管理服务**：管理服务运行在每台服务器JVM上，也运行在基本配置的应用程序服务器中。管理服务提供一些关键功能，这些功能可以操纵用于服务器及其组件的配置数据。配置在存储库中保存，也就是在服务器文件系统中保存一组XML文件。
- **会话数据库**：会话详细资料可以保存在中央会话数据库中，保证多用户环境下的会话持久化。承载特定应用程序的多个应用程序服务器共享数据库信息来运行状态组件的会话状态。
- **脚本客户端**：脚本客户端wsadmin增强对基于WEB的管理控制台的调节。这样管理员就可以使用命令行接口了。脚本客户端使用脚本帮助自动完成对多台应用服务器和节点的管理。


应用服务器是WebSphere的主要组件。他是在JVM上运行的，为应用程序代码提供运行库环境。

应用程序服务器包含执行特定Java应用程序组件的容器。应用程序服务器有三个容器：

- **WEB容器**：WEB容器处理servlet和JSP文件。它默认有一个会话管理器，在处理servlet时创建请求对象和响应对象。然后它访问servlet服务的方法。WEB容器处理嵌入式HTTP服务器、外部WEB服务器插件或WEB浏览器的HTTP请求。
- **EJB容器**：EJB容器为EJB提供运行库服务，处理对会话和实体bean的请求。企业bean位于EJB模块中。加载在应用服务器上的bean不直接与服务器交互。EJB容器提供EJB与服务器之间的接口。容器和服务器提供bean运行库环境。
- **JCA容器**：Java连接器结构（Java Connector Architecture，JCA）容器是WAS提供的组件。可以把从EIS提供商处购买的JCA资源适配器查到Java连接器结构上，用兼容JCA的应用程序来配置和使用。

嵌入式WebSphere JMS提供者依靠JMS服务器来实现集成的消息传递功能。它适合点到点和发布/注册类型的消息传递，是事务处理管理服务的一部分。JMS服务器支持消息驱动的bean以及WebSphere单元中的消息传递。在Base版本中，JMS服务器在与应用服务器相同的JVM中运行。

应用程序服务器JVM拥有名称服务，它提供Java命名和目录接口（JNDI）名称空间。该服务注册应用程序服务器所承载的所有EJB和J2EE资源（JMS、J2C、JDBC、URL和JavaMail）

应用程序服务器JVM也承载依赖配置库中安全设置的安全服务，以提供验证和授权功能。

在讨论WAS Base中，需要考虑有关应用程序的三个基本组件：

- **客户端应用程序容器**：是在客户端计算机上单独安装的组件。它使客户端能够在与EJB兼容的J2EE环境中运行应用程序。可以用可执行的命令行工具（launchClient）来启动客户端应用程序及其客户端容器运行库。
- **应用程序**：是唯一设计的，是由应用程序服务器承载并运行的。应用程序在存储到企业应用程序档案（EAR）之后，可以有一个或多个应用程序服务器共享应用程序。
- **应用程序数据库**：是在一个企业系统中的数据库服务器上运行的，在那里，堕胎应用程序服务器可以共享同一数据库。


WAS Base版包含若干个基于WEB的或与WEB相关的组件：

- **WEB服务器和WEB服务器插件**
- **嵌入式HTTP服务器**
- **WEB管理控制台和应用程序**
- **WEB服务引擎**：WEB服务引擎不能作为单独的物理组件存在。应用程序服务器为附加服务引进了一系列的API。WEB服务是作为配合J2EE应用程序的一组API提供的。WebSphere WEB服务引擎是机遇AXIS的，使用SOAP、WSDL、UDDL和WSIF规范。


### ND版

WAS ND配置支持多个节点，每个节点都有一个节点代理组件和多个应用程序服务器。都是在叫做单元的管理单元中运行的，单元在Deployment Manager中。你可以用Network Deployment单元配置负载平衡的应用程序服务器集群。DM管理单元中组件的配置和应用程序二进制文件都被分配给每个节点上的本地副本。

ND运行库结构包含多个组件：

- **单元**：单元是管理域中节点的集合。为了进行配置，每个节点都有名称。单元住配置库保存单元中每个节点的配置和应用程序二进制文件。存储库是通过DM管理的。
- **Deployment Manager**：DM组件是管理单元所有部分的唯一位置。DM承载基于WEB的管理控制台应用程序。它负责每个节点的存储库内容（配置和应用程序二进制文件）。
- **主配置库**：主配置库保存单元的配置数据。DM执行对它的更新。每个节点的配置库是主库的同步子集。节点库针对应用程序服务器的访问是只读的。
- **节点代理**：节点代理是执行下列功能的管理组件：文件传输服务、配置同步和性能监控。节点代理通过与DM的交互管理可管理的组件。节点代理配置同步化，位DM执行管理操作。它还与应用程序服务器和JMS服务器交互来管理每台服务器并更新其配置和应用程序二进制文件。
- **UDDI注册表**：WAS提供专用的UDDI注册表，这样企业就可以在组织内管理其自身的WEB服务了。也可以向其他组织或企业提供代理服务。管理员把UDDI注册表作为符合J2EE规范的WEB应用安装到利用起服务的没太应用程序服务器上。


WEB服务网关是在WEB服务调用期间链接Internet和企业内部网的中间组件。它管理：

- WEB服务
- WEB服务触发的过滤器
- 承载进出WEB服务的请求的通道
- 对在其中可以注册WEB服务的UDDI注册表的引用

	网关运行在WEB服务定义语言（WSDL）和WEB服务调用框架（WSIF）智商，以进行部署和调用。
	

Edge组件是承载WEB易访问的内容并提供Internet访问的一种有效而经济的方式。该软件一般是在邻近企业内部网和Internet之间边界的机器上运行的。Edge组件包含告诉缓冲代理器和负载平衡器，它们帮助管理员提高服务级别，以便具有访问权限的内部和外部用户能够更好地访问企业服务器计算机上的文档。

集群是为工作负载平衡而设计的应用程序服务器进程的逻辑排列。构成集群一部分的应用程序服务器是该集群的成员，在其上应部署相同的应用程序组件。不要求集群成员共享任何配置数据。

集群成员可以位于一个节点上（垂直群集），可以在多个节点上（水平群集），也可以是两种类型的混合。

被管理的服务器或被管理的进程是指构成WebSphere产品组件的所有系统进程。所有服务器都在管理域中占有一席之地。JMX支持是所有被管理进程的一部分。这些进程能够接收管理命令，在这些进程内分布有关被管理资源条件的管理信息。


WebSphere提供多项被管理的服务器或进程：

- Deployment Manager（ND）

	Deployment Manager是一个单元的所有配置信息和控制的单个访问点。

- 节点代理（ND）

	节点代理聚集并管理节点上所有WebSphere管理进程。
	
- 应用程序服务器（Base and ND）

	应用程序服务器是承载J2EE应用程序的托管服务器。
	
- JMS服务器（ND）

	JMS服务器是为节点承载嵌入式消息传递服务的托管服务器


### WebSphere组件共存

ND为实现多个WAS安装在一台机器上的共存而支持一系列的场景。

#### 多个WAS实例

WAS在一台机器上支持两种特定类型的运行库实例：

* 应用程序服务器，是WAS一个安装版本的多个实例。
* DM，是ND的一个安装版本的多个实例

#### 来自单一安装版本的多个服务器实例

WebSphere支持采用一个安装版本运行多个数据库实例。例如，你可以从一个WAS安装版本中开发并运行多个WAS实例。你也可以从独立的ND安装版本中创建并运行多个DM。

可以为耽搁ND安装版本配置多个DM，但是这些DM不能提供相互的故障转换，也不支持群集。

#### WAS和ND共存

你可以在与WAS相同的机器上安装ND软件。这样做有明显的优点。例如，你不需要专门的机器来承载单元DM和它的主单元存储库。这还能够重新使用为节点机器提供的当前备份工具。

但这样做也有缺点，如果WAS或DM出现了任何问题，则需要重建节点，需要重新部署其他组件。

#### 单个Web服务器与多个Web服务器

除了多个实例以外，当独立机器上有多个应用程序服务器共存时WAS提供Web服务器选项。这些选项包括：

- 在一台机器上共享多个版本的应用程序服务器的一台Web服务器
- 用于WAS的多个实例的一台Web服务器
- 在运行WAS多个实例时用于每个应用服务器实例的独立Web服务器


********


## Websphere拓扑结构考虑因素

在为Websphere不熟选择合适的拓扑结构或配置是，有七个重要因素需要考虑：

### 可用性

> 拓扑结构需要有合适的进程冗余，以避免单个故障点，从而最大化系统可用性。