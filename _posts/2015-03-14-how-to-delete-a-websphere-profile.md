---
layout: post
title: "How to delete a Websphere profile"
description: ""
category: "Middleware"
---
{% include JB/setup %}

How to delete a Websphere profile

#### 1. if you are removing a profile that is federated to a cell (Including ND Cell and AdminAgent)

	cd {AdminAgent Home}/bin
	deregisterNode -connType SOAP -port 8877 -profilePath "F:\IBM\Websphere\AppServer\profiles\AppSrv01" -username xxxx -password "Password"


**Attention:**

- Use removeNode command if it is a ND Cell. 
- Use deregisterNode command if it is a AdminAgent managed node. Here I use this secario as the example.
- Please ensure **The AdminAgent must be running**.
- If you don't remember the SOAP port of the AdminAgent, can check the serverindex.xml under {AdminAgent Home}\config\cells\THAIPWAPP15AACell01\nodes\THAIPWAPP15AANode01

<!-- more -->

The result should be like below:

![alt Deregister](http://dellyqiao.qiniudn.com/2015/01/08/deregister.png/scale)

#### 2. Stop all the JVM services in the profile you will be deleted. Try to ensure no java process is running from OS level.

	Windows: check from task manager.
	Linux: ps -ef | grep java


#### 3. Change directory to {WAS Home}/bin, then run the delete profile command:


	manageprofiles -delete -profileName AppSrv01

The output should be as below:

![alt delete profile](http://dellyqiao.qiniudn.com/2015/01/08/delete profile.png/scale)

#### 4. Clean the profile registry using the following command:


	manageprofile -validateAndUpdateRegistry


#### 5. Remove the profile folders.

Till now, most of the subfolders of the profile *AppSrv01* should have been removed, only *AppSrv01/log* folder still there. The last step is to delete the profile folder under {WAS Home}/profiles/.

