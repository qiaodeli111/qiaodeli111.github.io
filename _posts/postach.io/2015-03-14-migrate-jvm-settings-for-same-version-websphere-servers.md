---
layout: post
title: "Migrate JVM settings for same version Websphere Servers"
description: ""
category: ["Middleware", "Migration"]
---
{% include JB/setup %}


### Important

Only when your senario is exactly same as below, you may try this method to migrate the **Application Server** configurations, in other word, this method can only copy the **JVM** configurations.

This scenario is as below:

  * Your profile have the capability to create new application server. (For example, the ND cell member Node, the standard application server profile which is managed by AgentAdmin)
  * You want to copy the configuration to another server with the same version WAS installed. (both server installed WAS 8.0, for example)

<!-- more -->

### There are better solutions for below scenarios, I will write another essays for these situations:

  * to migrate the configures from old version to the new version
  * To migrate the configures for Standard / Base version WAS with only 1 Node.

### Steps

For example, we'd like to migrate JVMs from server APP11 to APP15:

#### 1\. Open wsadmin with jython language on the APP11 :
    
    cd {Profile Home}\bin
    wsadmin -lang jython -username wasadmin -password "Password**"
    

Output should be as below:

![alt JVMlist](http://dellyqiao.qiniudn.com/2015/01/08/wsadmin.png/scale)

**Take Note!**

Ensure you saw the information like: you have successfully connected to the profile which includes the JVMs you want to export. **Do not** run the command under AdminAgent/bin, this way will cause exception.

#### 2\. Export the old JVM configuration:
    
    
    wsadmin>AdminTask.exportServer('[-nodeName THAIPWAPP11Node02 -serverName ams-server -archive F:/IBM/Archive/amsArchive.car]')
    

Here I exported 4 JVM configuration archives:
![alt JVMlist](http://dellyqiao.qiniudn.com/2015/01/08/export1.png/scale)

![alt JVMlist](http://dellyqiao.qiniudn.com/2015/01/08/export2.png/scale)

**Take Noted!**

The archive file path must be with `/` even you are running this command in Windows environment.

#### 3\. Create the new profiles, and register the application profile into DMGR or AdminAgent.

  1. Create the profiles. Here I will create `AppSrv01` and will register it into `AdminAgent01`.

	![alt JVMlist](http://dellyqiao.qiniudn.com/2015/01/08/profiles.png/scale)

  2. Change directory to AdminAgent01 bin folder, run registerNode command:
    
        cd F:\IBM\WebSphere\AppServer\profiles\AdminAgent01\bin
        registerNode -profileName AdminAgent01 -host APP15 -profilePath "F:\IBM\WebSphere\AppServer\profiles\AppSrv01" -connType SOAP -port 8877 -username wasadmin -password "Password**" -nodeusername wasadmin -nodepassword "Password**"
    

	Ensure the output information dosn't contains any error. 

	![alt JVMlist](http://dellyqiao.qiniudn.com/2015/01/08/registernode.png/scale)

  3. Logon AdminAgent IBM Console to verify the result.

	![alt JVMlist](http://dellyqiao.qiniudn.com/2015/01/08/nodeselect.png/scale)

#### 4\. Import the JVM into the new environment:

Now your WAS profile have the capacity to create application servers as below:

![alt JVMlist](http://dellyqiao.qiniudn.com/2015/01/08/createserver.png/scale)

To import the configuration archive files, WAS will create the application server with the congiurations automatically.

Copy the .car files into F:\IBM\Archive\ of APP15. Then execute below commands:
    
    
    cd {AppSrv01 Home}\bin
    wsadmin -lang jython -username wasadmin -password "Password**"
    //after logged in wsadmin tool:
    AdminTask.importServer('[-archive F:\IBM\Archive\healthcareArchive.car -nodeInArchive APP11Node01 -serverInArchive HealthCare-server -nodeName APP15Node02 -serverName HealthCare-server]')
    AdminConfig.save()
    

Here I imported 4 JVMs, and then saved the configuration with `AdminConfig.save()` together:

![alt JVMlist](http://dellyqiao.qiniudn.com/2015/01/08/importresult.png/scale)

#### 5\. Verify the result from IBM Console:

![alt JVMlist](http://dellyqiao.qiniudn.com/2015/01/08/JVMlist.png/scale)

You may open Process Defination settings to verify if the configuration is same as the old JVMs.

### Configurations that will not be copied

Frankly, this is not a full configuration copy, even you imported JVM via this method, some parameters still need to be changed manually. And you will still need to create JAAS/DataSource/JMS manually. Those configurations will not be copied out.

I'm not exactly know the full list of the parameters that need to be modified manually, and I will keep updating the list here once I noticed any:

* The JVM port. You need to modify them manually.

	![alt JVMport](http://dellyqiao.qiniudn.com/2015/01/08/JVMport.png/scale)
	
* Application Server Resources

	You may created some resources (such as JDBC provider, DataSource) under the application server scope. For this case, the related resources should also be copied out with the application servers.

	But as per my experience, sometimes some of the resources will not be exported into the archive files normally, sometimes the resources of server A will be exported into the archive of server B......

	It is strange, thus we need to manually check all the resources by ourselves.
	

### References

Here I attache the reference article from IBM and you may get more information from there:

<http://www-01.ibm.com/support/knowledgecenter/SS7JFU_8.0.0/com.ibm.websphere.express.doc/info/exp/ae/rxml_atconfigarchive.html>

Below attach the usage of importServer:
    
    
        Parameters and return values
    
        -archive
            Specifies the fully qualified path of the configuration archive. (String, required)
        -nodeInArchive
            Specifies the node name of the server defined in the configuration archive. (String, optional if there is only one node defined in the configuration archive, required if there are multiple nodes defined in the configuration archive)
        -serverInArchive
            Specifies the name of the server defined in the configuration archive. (String, optional if there is only one server defined on the specified nodeInConfiguration archive, required if there are multiple servers defined under the specified nodeInConfiguration archive)
        -nodeName
            Specifies the node name where the server is imported. (String, optional if there is only one node)
        -serverName
            Specifies the server name where the server is imported. If the server name that you specify matches an existing server name under the node, an exception is created. (String, optional, default: serverInArchive)
        -coreGroup
            Specifies the core group name to which the server should belong. (String, optional) 
    
        AdminTask.importServer('[-archive c:/myServer.car -nodeInArchive node1 -serverInArchive server1]')
    
        AdminTask.importServer('[-archive F:\IBM\Archive\iPOSArchive.car -nodeInArchive THAIPWAPP11Node01 -serverInArchive iPOSAppSrv -nodeName THAIPWAPP15Node01 -serverName iPOSAppSrv]')
    
