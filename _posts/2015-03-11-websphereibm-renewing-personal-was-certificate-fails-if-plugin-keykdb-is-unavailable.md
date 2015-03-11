---
layout: post
title: "解决Websphere内置证书更新失败——IBM Renewing personal WAS certificate fails if plugin key.kdb is unavailable"
description: ""
category: "Middleware"
---
{% include JB/setup %}

最近Websphere服务器集群遇到一个问题，在用部署脚本部署应用的时候出现错误，显示SSL证书过期导致无法建立安全连接。

打开IBM Console，发现Console已经无法识别那个Node，Node状态显示为“？”，Node下所有的应用服务器的状态也无法被Console所识别。重启Nodeagent和DMGR均无效果。

为什么会这样呢，经过检查，我发现那个Node中的内建证书过期了，可以在Security -> SSL Certificate and Key Management -> Key Stores and certificates -> NodeDefaultKeyStore -> Personal Certificates 这个路径下查看。

<!-- more -->


有一个情况也是我注意到的，这个集群下有几个Node，但是唯独其中的那一个Node的证书更新失败。当我尝试去手动更新证书的时候，发现证书更新错误：

	An error occurred renewing default: 	
	com.ibm.websphere.crypto.KeyException: KeyStore "C:/WebSphere/AppServer/profiles/Dmgr01/wstemp/1623776755/workspace/cells/winwas70dCell01/nodes/winwas70dNode01/servers/webserver1/plugin-key.kdb" does not exist.


在查找IBM资料和找IBM确认之后，确定恢复的办法如下：

1）删除CMSKeyStore下的缺失证书的定义。

2）勾选过期的证书，点击“renew”按钮。

3）重启所有有问题的节点的服务，顺序：停止APP Server，Nodeagent， 然后启动Nodeagent，启动APP Server。


**注意：这种方法无论在证书过期前还是过期后都是有效的，即使Node的状态是一个“？”也没关系。只要确定DMGR所在的服务器跟Nodeagent所在服务器之间的相关端口都开启了就行（最主要的应该是SOAP端口，Nodeagent默认为8878，可以在DMGR服务器上执行：telnet nodeagentserver 8878 去验证一下）。**


IBM support center的资料原文如下，里面有证书无法更新的原因，供参考：

## Technote (troubleshooting)

<div markdown="1" style="font-family: Arial, sans-serif; font-size: 13px; line-height: 1.2; text-indent: 0pt; word-spacing: normal; background-color: rgb(255, 255, 255); border: 0px rgb(0, 0, 0); clear: both; cursor: auto; direction: ltr; float: none; list-style: disc outside none; margin: 0px; outline: invert none 0px; table-layout: auto; vertical-align: baseline; visibility: inherit; z-index: auto; background-position: 0% 0%;">

  


## Problem(Abstract)

When the personal certificate in your WAS NodeDefaultKeyStore or CellDefaultKeyStore expires, you need to renew it (either manually or automatically using the expiration monitor).  
This might fail with an error message, that the CMSKeystore is not available.

## Symptom

<div markdown="1" style="border: 0px rgb(0, 0, 0); cursor: auto; direction: ltr; float: none; line-height: 1.2; list-style: disc outside none; margin: 0px; outline: invert none 0px; table-layout: auto; text-indent: 0pt; vertical-align: baseline; visibility: inherit; word-spacing: normal; z-index: auto; background-position: 0% 0%;">

The expiring certificate is not renewed in the NodeDefaultKeyStore, although the ISC already shows the new personal certificate.

Steps to recreate:

  1. Login to the AdminConsole (ISC)
  2. Go to Security -> SSL Certificate and Key Management
  3. Key Stores and certificates -> NodeDefaultKeyStore -> Personal Certificates
  4. select the personal certificat that you want to renew (normally it's called "default")
  5. Press the "Renew" Button
  6. If the plugin-key.kdb cannot be accessed by the DMgr, you will an error similar to this:  
  
An error occurred renewing default: com.ibm.websphere.crypto.KeyException: KeyStore "C:/WebSphere/AppServer/profiles/Dmgr01/wstemp/1623776755/workspace/cells/winwas70dCell01/nodes/winwas70dNode01/servers/webserver1/plugin-key.kdb" does not exist.  
  
Along with a CWPKI0033E error in the DMgr's SystemOut.log, telling the same problem.
  7. Despite of the error above, **the** **list of certificates will already show you the new certificate with the new expiration date, but this is a false notification!**  

  
If you logout and login again, you will find that there is still the old personal certificate in the NodeDefaultKeyStore!</div>

## Cause

<div markdown="1" style="border: 0px rgb(0, 0, 0); cursor: auto; direction: ltr; float: none; line-height: 1.2; list-style: disc outside none; margin: 0px; outline: invert none 0px; table-layout: auto; text-indent: 0pt; vertical-align: baseline; visibility: inherit; word-spacing: normal; z-index: auto; background-position: 0% 0%;">

The WebServer Plugin needs the signer certificate from the Node's personal certificate to ensure a secure Plugin-WAS connection.

  
If the plugin-key.kdb ist not available at the defined location of the CMSKeyStore (or not accessible due to file permission problems), then the automatic signer exchange is not possible for the Deployment Manager - hence the renewal of the certificate cannot complete and is interrupted.

</div>

  


## Resolving the problem

<div markdown="1" style="border: 0px rgb(0, 0, 0); cursor: auto; direction: ltr; float: none; line-height: 1.2; list-style: disc outside none; margin: 0px; outline: invert none 0px; table-layout: auto; text-indent: 0pt; vertical-align: baseline; visibility: inherit; word-spacing: normal; z-index: auto; background-position: 0% 0%;">

You either need to ensure, that all keystores and truststores which are defined in the cell are accessible, including the CMSKeyStore for the plugin.  
If for some reason the plugin-key.kdb has been removed and is no longer required in this cell, the CMSKeyStore definition should also be deleted in the list of keystores in the ISC.

</div>

</div>
