---
layout: post
title: "如何从Symantec为自己的网站购买SSL证书"
description: ""
category: Security
tags: []
---
{% include JB/setup %}


当我们的网站有加密会话的需求的时候，我们会把所有的请求从HTTP协议转向HTTPS协议。使用HTTPS协议加密session的话我们就需要用到安全证书。当然我们可以自己生成安全证书，但是没人能证明你的安全证书本身足够安全。所以，我们需要从业界公认的证书提供商处去购买一个安全性强的证书，让他们来证明你的网站的会话确实可以被加密。。。。。。

基于这样的需求，有实力的公司会选择信誉良好的证书提供商，以证明他们的网站确实很安全（在访问银行类保险公司类的网站的时候基本上每一个网页都会有HTTPS加密，浏览器的地址栏旁边会提示说安全证书哪家最强云云......），价格问题会放在其次。

Symantec就是其中一个比较不错的证书提供商。但是他的网站的易用性确实不怎么好，很多时候申请的信息会莫名其妙的自动被修改。这篇文章是基于我自己的大量的证书申请经历创建的，按照本文的方法做，保你申请无忧。

<!-- more -->
本文以申请IBM HTTP Server的证书为例。另外Symantec网站的证书申请流程也已经经过了数次变更，目前的步骤仅适用于现在的情况。具体步骤如下：

## 1. 创建证书申请CSR文件

IBM HTTP Server（IHS）的证书由几部分组成：

- kdb文件：用来存放安全证书的容器，存放公钥和私钥
- cer文件：证书文件，也就是从证书提供商（以后简称CA —— Certificate Authority 身份认证机构，本文特指Symantec）得到的个人证书，私钥
- csr文件：证书request文件，申请证书时得告诉CA你需要的是什么样的证书，那些信息就放在这个文件里
- sth文件：kdb是需要设置密码的，这个文件就是用来放置密码的

注意，除了kdb文件和sht文件之外，其他文件的扩展名都是任意的，csr,arm,txt统统没问题，关键的是文件的内容要符合规范需求。

下面介绍如何生成CSR文件：



IHS证书使用IHS自带的ikeyman去创建。打开{IHS Home}/bin/ikeyman，然后点击新建：

![1-1](http://dellyqiao.qiniudn.com/2015/02/04/1-1.png/scale)

继续，为这个KDB设置密码，记住勾选将密码存储到文件：

![1-1](http://dellyqiao.qiniudn.com/2015/02/04/1-2.png/scale)

密码设置完之后就能进入个人证书请求的界面了。选中个人证书请求项：

![1-1](http://dellyqiao.qiniudn.com/2015/02/04/1-3.png/scale)

**最前面的四项是最重要的。** 

- 密钥标签：为即将申请到的个人证书起一个名字，如果你打算在这个KDB中放置多个证书，那就需要认真起名，便于以后在配置文件中能正确地设置。
- 密钥大小：定义密钥的大小，2048是一个不错的选择。
- 签名算法：这里看个人的需求。现在基本上都需要申请SHA256加密算法的证书，因为如果你选择SHA128，新版本的Chrome浏览器会认为你的网站加密算法太弱，不够安全。既然钱花到了，自然得买个好的。
- 共用名：这里要填写你的网站的域名。这儿要跟网站域名严格匹配，否则你的网站将会出现证书与域名不匹配的警告。

其他项目根据自己的情况填写就好。

![1-1](http://dellyqiao.qiniudn.com/2015/02/04/1-4.png/scale)

保存过后会生成一个certreq.arm文件，用任何编辑器打开这个文件即可看到里面的内容。把里面的内容拷贝出来。

![1-1](http://dellyqiao.qiniudn.com/2015/02/04/1-5.png/scale)


## 2. 在Symantec网站上购买证书

首先，登录Symantec Trust Center网站，[https://trustcenter-enterprise.websecurity.symantec.com](https://trustcenter-enterprise.websecurity.symantec.com)

登录之后按照提示添加要购买的证书的域名相关的公司信息。

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/2-1.png/scale)

填写下图中的所有项目，最终要的项目是“Organization Information”和“Organization Contact”。注意：Organization公司名一定要填写真实完整的公司名，要填写在工商局注册的名字，Symantec自己有一个数据库，他们会去验证你的公司是否真实有效。

联系人也要填写能联系到的人，在购买新证书的时候Symantec的支持团队会打电话给这个人确认你的订单。

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/2-2.png/scale)

填完确定后就能看到刚刚填写的信息了。现在继续在"Domains authenticated"里添加跟这个“Organization”相关的域名。

你也可以看到“Organizations”是一个列表，不止可以填一个。像我这样的工作性质，作为乙方，位甲方去购买证书的情况，这个设置是非常有用的，因为我的甲方有很多个国家的很多分公司，每个分公司的联络人都不一样。

在“Domains Authenticated”里面填写好之后，比如我的“Organizations”是dellyserver，授权的域名包含了www.dellyserver.com，那么当我购买www.dellyserver.com的安全证书的时候就会直接调用我在之前填写的“dellyserver”公司的信息（包括地区，联系人，公司名，等等）

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/2-3.png/scale)

上面的部分设置好之后就可以开始获取证书了：

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/2-4.png/scale)

选择一个证书的类型。每种证书的价格都是不一样的，根据个人需求选择需要的证书。一般选择“Secure Site Pro”就OK：

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/2-5.png/scale)

接下来就要用到我们的CSR了。“Server Type”中选择要购买的证书类型。我们需要IHS证书，所以选择Other。接下来把CSR文件里的内容粘贴到下面的框里。点击下一步。

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/2-6.png/scale)

出现了一个警告信息，然后在右侧也出现了“Common Name”，说明证书申请已经被识别出来了。警告信息可以无视，重新点击下一步。

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/2-7.png/scale)

接下来到达了订单Summary，注意看好各项信息是否正确。

注意：这里的“Location” “Country”这些项目有时候不会跟你的CSR申请完全一致。比如如果你的Symantec账户注册的时候选择的所在地是香港，而之前在设置Organization的时候也没有完全设置正确，那么我们在位泰国的这家公司购买证书的时候，这个City、Location就会很有可能变成中国。如果出现这种情况，也可以先下单，之后给Symantec的Support团队发邮件告诉他们做一下修正。

（根据之前的经验，之前有段时间就算你的Organization设置正确了，这个location也会出问题。）

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/2-8.png/scale)

确认订单之后新买的证书就会出现在自己的SSL证书库中了。

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/2-9.png/scale)


这时Symantec网站上的相关操作就完成了，等待Symantec打电话核实订单就OK。


## 3. 获取并导入证书

在经过几天的等待之后（一般两个工作日就差不多了），你将会收到Symantec发来的邮件：

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/3-1.png/scale)

我们的证书已经准备好了！所以，点开邮件中的连接，然后点击“Get started”:

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/3-2.png/scale)

进入到下载证书的界面，不管之前申请的证书是什么类型，现在都可以下载全类型的证书文件。（这个功能也是前段时间改版后才出现的）

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/3-3.png/scale)

我们选择IHS，然后点击“Download”：

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/3-4.png/scale)

下载到的是一个压缩包，里面包含简要说明（getting_started.txt），个人证书（ssl_certificate.arm），Symantec提供的G4证书（IntermediateCA.arm），还有Symantec的G5证书（crossRootCA.arm）。

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/3-5.png/scale)

用ikeyman打开我们之前创建的KDB文件，选择“Personal Certificates”项目，然后点击右侧的“Receive”按钮

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/3-6.png/scale)

选择“ssl_certificate.arm”，并且点击确定：

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/3-7.png/scale)

这时将会看到在“Personal Certificates”项目下多出来一条“* xxxxxxx”的项目，这个就说明我们得到的个人证书已经正常导入到证书里了。也可以惦记个人证书申请项目，会发现申请的条目会自动消失。

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/3-8.png/scale)

接下来选择“Signer Certificates”签署人证书项目，这个项目是用来放置CA提供的公钥证书的。

进入此项目之后，点击“Add”，分别选择我们得到的G4和G5证书，在取名的时候可以写G4和G5.

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/3-9.png/scale)

添加G4和G5之后，CA证书还是不够，继续点击右侧的“Populate”按钮，把所有IHS的ikeyman工具内置的CA证书全部导入到KDB文件中。照图操作即可：

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/3-10.png/scale)

最后CA证书导入完成，这个证书也就完全做好了。接下来就是修改HTTP Server的配置文件，引入这个KDB文件即可。

![symantec](http://dellyqiao.qiniudn.com/2015/02/04/3-11.png/scale)


----

全文到此结束。如果在过程中有任何问题，也可以直接联系我进行讨论。