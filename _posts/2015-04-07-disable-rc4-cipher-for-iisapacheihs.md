---
layout: post
title: "Disable RC4 Cipher for IIS/Apache/IHS"
description: ""
category: ["Security", "Middleware"]
---
{% include JB/setup %}

最近多家安全网站曝出网站系统使用TLS安全协议时如果使用RC4 Cipher会有安全问题，如下文所述：

[http://www.imperva.com/docs/HII_Attacking_SSL_when_using_RC4.pdf](http://www.imperva.com/docs/HII_Attacking_SSL_when_using_RC4.pdf)

[http://securityaffairs.co/wordpress/35352/hacking/bar-mitzvah-attack-on-rc4.html](http://securityaffairs.co/wordpress/35352/hacking/bar-mitzvah-attack-on-rc4.html "http://securityaffairs.co/wordpress/35352/hacking/bar-mitzvah-attack-on-rc4.html") 

[http://www.securityweek.com/new-attack-rc4-based-ssltls-leverages-13-year-old-vulnerability](http://www.securityweek.com/new-attack-rc4-based-ssltls-leverages-13-year-old-vulnerability "http://www.securityweek.com/new-attack-rc4-based-ssltls-leverages-13-year-old-vulnerability")


对于常见的Web服务器来说，要修复这个安全漏洞，需要禁用RC4 Cipher。下面列出几种服务器中禁用RC4 Cipher的方法：

<!-- more -->

### 1) IIS

IIS服务器使用哪种安全协议，安全协议使用哪种密钥都是需要在注册表里去配置的。RC4 Cipher也不例外，在Windows Server 2003中的配置方法如下：

	How to Completely Disable RC4 
	Clients and Servers that do not wish to use RC4 ciphersuites, regardless of the other party's supported ciphers, can disable the use of RC4 cipher suites completely by setting the following registry keys. In this manner any server or client that is talking to a client or server that must use RC4, can prevent a connection from happening. Clients that deploy this setting will not be able to connect to sites that require RC4 while servers that deploy this setting will not be able to service clients that must use RC4. 
	
	[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC4 128/128] 
	"Enabled"=dword:00000000
	[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC4 40/128] 
	"Enabled"=dword:00000000
	[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC4 56/128] 
	"Enabled"=dword:00000000

参考下面链接：

[http://blogs.technet.com/b/srd/archive/2013/11/12/security-advisory-2868725-recommendation-to-disable-rc4.aspx](http://blogs.technet.com/b/srd/archive/2013/11/12/security-advisory-2868725-recommendation-to-disable-rc4.aspx "http://blogs.technet.com/b/srd/archive/2013/11/12/security-advisory-2868725-recommendation-to-disable-rc4.aspx")


### 2) IHS

IHS即IBM HTTP Server，所有的配置都在httpd.conf中，所以修改httpd.conf就好了。

默认情况下，IHS是不会启用RC4的，但是由于前几年各界都比较认可RC4，所以一般的IHS服务器都启用了RC4。想要禁用的话删掉对应的配置就好了。

首先我们要找到相关的SSL的配置，并且确定哪些配置是用来启用RC4的。可能的配置是这样的：

	SSLCipherSpec SSL_RSA_WITH_3DES_EDE_CBC_SHA
	SSLCipherSpec SSL_RSA_WITH_RC4_128_SHA
	SSLCipherSpec SSL_RSA_WITH_RC4_128_MD5
	SSLCipherSpec SSL_RSA_WITH_DES_CBC_SHA
	SSLCipherSpec TLS_RSA_EXPORT1024_WITH_RC4_56_SHA
	SSLCipherSpec TLS_RSA_EXPORT1024_WITH_DES_CBC_SHA

也有可能是这样：

	SSLCipherSpec 34
	SSLCipherSpec 35
	SSLCipherSpec 3A

如果是启用了34、35或者3A这样的类似代码一样的东西，我们如何确定哪个是跟RC4相关的呢？

请参考如下链接：

[http://publib.boulder.ibm.com/httpserv/ihsdiag/ssl_questions.html#check_ssl_config](http://publib.boulder.ibm.com/httpserv/ihsdiag/ssl_questions.html#check_ssl_config "http://publib.boulder.ibm.com/httpserv/ihsdiag/ssl_questions.html#check_ssl_config")


	How can I check the end result of my SSL configuration changes to verify what prototcols and ciphers my server will use ?
	In IHS 8.0 and later:
	
	If there is a question of how your configuration customizations are ultimately combined, you can run
	 apachectl -t -DDUMP_SSL_CONFIG
	to see the effective values after processing the configuration.
	 
	Example output of the default configuration + SSLEnable is:             
	                                                                        
	SSL server defined at:                                                  
	/home/ihs/2.2/built/conf/httpd.conf:907                       
	Server name: 127.0.1.1:443                                              
	SSL enabled: YES                                                        
	FIPS enabled: 0                                                         
	Keyfile: /home/ihs/IHSJTest/data/ihskeys/key.kdb                
	Protocols enabled: SSLV2,SSLV3,TLSv10,TLSv11,TLSv12                     
	Ciphers for SSLV2: (defaults)                                           
	Ciphers for SSLV3: (defaults)                                           
	TLS_RSA_WITH_AES_128_CBC_SHA(2F),TLS_RSA_WITH_AES_256_CBC_SHA(35b),S    
	SL_RSA_WITH_RC4_128_SHA(35),SSL_RSA_WITH_RC4_128_MD5(34),SSL_RSA_WIT    
	H_3DES_EDE_CBC_SHA(3A)                                                  
	Ciphers for TLSv10: (defaults)                                          
	TLS_RSA_WITH_AES_128_CBC_SHA(2F),TLS_RSA_WITH_AES_256_CBC_SHA(35b),S    
	SL_RSA_WITH_RC4_128_SHA(35),SSL_RSA_WITH_RC4_128_MD5(34),SSL_RSA_WIT    
	H_3DES_EDE_CBC_SHA(3A)                                                  
	Ciphers for TLSv11: (defaults)                                          
	TLS_RSA_WITH_AES_128_CBC_SHA(2F),TLS_RSA_WITH_AES_256_CBC_SHA(35b),S    
	SL_RSA_WITH_RC4_128_SHA(35),SSL_RSA_WITH_RC4_128_MD5(34),SSL_RSA_WIT    
	H_3DES_EDE_CBC_SHA(3A)                                                  
	Ciphers for TLSv12: (defaults)                                          
	TLS_RSA_WITH_AES_128_GCM_SHA256(9C),TLS_RSA_WITH_AES_256_GCM_SHA384     
	(9D),TLS_RSA_WITH_AES_128_CBC_SHA256(3C),TLS_RSA_WITH_AES_256_CBC_S     
	HA256(3D),TLS_RSA_WITH_AES_128_CBC_SHA(2F),TLS_RSA_WITH_AES_256_CBC     
	_SHA(35b),SSL_RSA_WITH_3DES_EDE_CBC_SHA(3A) 
	
	                            
	(Note: we report SSLv2 as enabled with zero available ciphers as an     
	implementation quirk)                                     


找到启用RC4相关的`SSLCipherSpec`之后，删除掉他们然后重启IHS服务就好了。


### 3) Apache

如链接：

[https://raymii.org/s/tutorials/Strong_SSL_Security_On_Apache2.html](https://raymii.org/s/tutorials/Strong_SSL_Security_On_Apache2.html "https://raymii.org/s/tutorials/Strong_SSL_Security_On_Apache2.html")

参考这一部分：

The recommended cipher suite:

	SSLCipherSuite AES128+EECDH:AES128+EDH

The recommended cipher suite for backwards compatibility (IE6/WinXP):

	SSLCipherSuite ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4
