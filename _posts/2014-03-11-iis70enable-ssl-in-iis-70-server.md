---
layout: post
title: 在IIS7.0中配置自签名证书(Enable SSL in IIS 7.0 Server)
description: ""
category: "Middleware"
---
{% include JB/setup %}


在IIS Manager界面，点击左侧的hostname。然后点击 Create Self-Signed Certificate。

![](http://dellyqiao.qiniudn.com/2015/03/111.png/scale)

右击 "Default Web Site”，然后点击 "Edit Bindings".

![](http://dellyqiao.qiniudn.com/2015/03/112.png/scale)

点击 “Add”，把HTTPS协议跟生成的self signed certificate绑定起来。

![](http://dellyqiao.qiniudn.com/2015/03/113.png/scale)

添加完成后点击“OK”，然后“Close”保存配置。

![](http://dellyqiao.qiniudn.com/2015/03/114.png/scale)

完成。