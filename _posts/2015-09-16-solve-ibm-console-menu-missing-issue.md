---
layout: post
title: "Solve IBM Console menu missing issue"
description: ""
category: "Middleware"
---
{% include JB/setup %}

1. Stop dmgr

		#under dmgr/bin: 
		./stopManager.sh -username <user_name> -password <password>

2. Clear these folders

		<dmgr_profile_root>/temp
		<dmgr_profile_root>/wstemp
		<dmgr_profile_root>/config/temp
 
3. Under /usr/IBM/WebSphere/AppServer/bin
		
		./iscdeploy.sh -restore
 
4. Restart dmgr and see if everything is alright
		
		./startManager.sh
 
5. If problem still persists, please run

		<dmgr_profile>/bin/wsadmin.sh -conntype NONE -f deployConsole.py remove
		<dmgr_profile>/bin/wsadmin.sh -conntype NONE -f deployConsole.py install
 
6. Restart dmgr to test.