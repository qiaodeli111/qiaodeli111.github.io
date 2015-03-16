---
layout: post
title: "解决IIS问题：HttpException (0x80004005): The current identity does not have write access to IIS Temporary ASP.NET Files.]"
description: ""
category: "Middleware"
---
{% include JB/setup %}

IIS Log中出现这个错误：

[HttpException (0x80004005): The current identity (NT AUTHORITY\NETWORK SERVICE) does not have write access to 'C:\Windows\Microsoft.NET\Framework\v2.0.50727\Temporary [ASP.NET](http://ASP.NET) Files'.]  
System.Web.HttpRuntime.SetUpCodegenDirectory(CompilationSection compilationSection) +9022990  
System.Web.HttpRuntime.HostingInit(HostingEnvironmentFlags hostingFlags) +152  

<!-- more -->




1） 进入.NET 安装目录</div>

	C:\Windows\Microsoft.NET\Framework\v2.0.50727

2） 执行此命令：</div>


	aspnet_regiis -ga IUSR_<machinename>
