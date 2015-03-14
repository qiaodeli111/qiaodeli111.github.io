---
layout: post
title: "在Apache中配置自签名证书(Self signed Certification in Apache)"
description: ""
category: "Security"
---
{% include JB/setup %}

1. Create a self-signed certificate to enable SSL  

		Under /etc/apache2/  
		openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./ssl.key/<hostname>.key -out ./ssl.crt/<hostname>.crt
	
2. under /etc/apache2/vhost.d/  

		cp vhost-ssl.template vhost-ssl

3. Edit vhost-ssl.conf, add below 2 lines in the <virtualhost> configuration:

	SSLCertificateFile /etc/apache2/ssl.crt/<hostname>.crt  
	SSLCertificateKeyFile /etc/apache2/ssl.key/<hostname>.crt
