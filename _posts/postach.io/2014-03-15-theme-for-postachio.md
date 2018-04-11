---
layout: post
title: "Theme for Postach.io - 适用于中国大陆postach.io用户的博客主题"
description: ""
category: "Tools"
---
{% include JB/setup %}

### 要干嘛
如果你在使用postach.io用evernote发博客，就一定有想法定制一下初始的主题，如果你发现不管怎么定制怎么换博客的速度依然很慢，那请试试我发布的这个主题吧。

这几天把自己的博客主题大翻修了一下，访问速度可以算是大幅提升了，因此就想把这个主题共享出来供大家使用，对所有的大陆用户来说，这绝对是能减少你工作量和头疼度的最好选择。

<!-- more -->


### 地址
地址在这儿：[https://github.com/qiaodeli111/theme-for-postach.io.git](https://github.com/qiaodeli111/theme-for-postach.io.git)

### 主题的替换方法
Step 1）从Github中把我的主题fork到你自己的repository中。

Step 2）
登录postach.io >> 编辑站点 >> Edit Theme >> 绑定Github >> 从下拉列表中选择 theme-for-postach.io >> 应用

### 截图

自适应桌面设备和移动设备：

![alt Large](http://dellyqiao.qiniudn.com/2015/01/02/screenshot-s.png)

![alt Small](http://dellyqiao.qiniudn.com/2015/01/02/screenshot-l.png)



### 关于本主题

本主题基于postach.io出品的默认主题[theme-original](https://github.com/postachio/theme-original.git)制作而来，采用BootStrap3.3.0搭建（原主题使用2.3.1，实在是已经过时了），利用BootStrap的响应式布局使得本主题自适应桌面设备和移动设备。请放心使用。

### 应用到自己的博客

Fork 这个项目到自己的项目列表中，然后在postach.io的站点管理功能里选择此主题就可以应用这个主题啦。

### 关于主题的使用

1. 主题已添加百度分享按钮和有言评论系统，如有需要，请在theme.html中找到相关的部分，替换相关代码即可。
2. 关于归档。postach.io实际上不提供归档功能，但是提供标签功能。通过标签为文章归档，点击标签也可以获得所有在该标签分类下的文章列表。
3. 静态资源的分离是博客站点提速的关键。对于静态资源库，比如图片资源，CSS资源和JS资源，我个人是把它们都放到了七牛CDN上，你有以下选择：

**公共资源（JQuery，BootStrap等）**

- 直接使用我的七牛CDN公共连接。我不保证所有的公共连接永久生效。
- 自己找国内的CDN托管库，比如又拍云CDN，百度CDN等等。

**个人静态资源（比如个人定制的style.css和个人头像的图片）**

- 申请一个CDN平台，然后修改好放到自己的CDN平台上，通过静态外链URL引用（**推荐**）
- 放到postach.io的平台上，通过主题标签引用（引用方法会在theme.html中提供——theme.html 18行和252行）（默认）


### 关于修改调试

Postach.io相较于github的一个好处就是发布的页面不必等30分钟以后才生效，主题资源即时同步到Github，效果即时同步。所以虽说不能搭建本地调试环境，但是终归调试起来还算方便。

### 为什么要发布本主题

对于国外的postach.io用户，本主题基本没有意义（除非有人赞同我的审美取向：））。对于在长城墙的国内用户来说，主要可以解决两个问题：

- 速度慢
- 不够大


#### 1）速度问题的由来
theme-original主题留给用户的可定制空间比较少，他的初衷是为了解耦，公共部分如果需要修改，只需修改一次所有用户都能受益，而且只把跟用户相关的部分暴露出来，也减少用户的学习成本，减少出问题的几率。

但是问题是本主题的公共静态资源库（比如CSS，JS）全都托管在国外的服务平台上，比如Google，Amazon云（postach.io本身好像就是托管在Amazon云平台上的），而且你还没得选。这样一来在访问自己站点的时候速度慢得不是一点点了。

比如：JQuery的引用，原主题给出的可供修改的主题页面只有theme.html，在其中使用了{{ meta_footer }}标签，主题读取这个标签后，可以得到JQuery的引用和一些有特殊作用的脚本，而作者所引用的JQuery库是托管在GoogleAPI上的。如果用这样的主题，国外用户没什么感受，国内的用户会看到页面载入的小圈会一直转一直转，因为国内压根就连不上googleapi。

#### 2）不够大
这个问题纯属个人喜好，个人希望自己的博客网站整体上简洁一点，字体稍微大一点，每一页显示的字数稍微多一点，页面利用率稍微高一点，当然了，功能也不能缺失。原主题只满足简洁的特点，其他特点都需要自己改动。


### 联系作者
任何问题请联系我：

- [博客网站](http://blog.dellyqiao.com)上留言
- Github项目留言
- 通过邮箱联系到我：qiaodeli111@126.com