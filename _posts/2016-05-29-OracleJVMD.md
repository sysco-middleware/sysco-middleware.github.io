---
layout: post
title: Java Virtual Machine Diagnostics (JVMD) Installation and Configuration on OEM Cloud Control 12C
categories: WebLogic
tags: [Oracle Enterprise Manager Cloud Control 12C, Java Virtual Machine Diagnostics, JVMD, OEM, Weblogic]
author: raul
---

# Installing JMVD
The aim of this guide is to show you how to activate the JVMD on OEM Cloud Control 12C. According to [1] these are the main steps to install JVMD

1.	Select the option Setup>Middleware Management>Application Performance Management

	![](/images/2016-05-29-OracleJVMD/01.png)

2.	Select this option Add>JVM Diagnostics Engine

	![](/images/2016-05-29-OracleJVMD/02.png)

3.	In this step this information is filled: 

	Information about JVM engine, this information will be used to create a new Weblogic managed server where the JVM engine will be deployed. This new managed server will be part of the GC Domain.

	•	Host	
	•	Managed Server Name		
	•	Managed Server Listen Port	
	•	Managed Server SSL Listen Port

	Host credentials, credentials for the host where Oracle Management Services (OMS) is running
	
	Weblogic domain credentials, to specify the credentials of the GC Domain’s weblogic user.

	![](/images/2016-05-29-OracleJVMD/03.png)
	
4.	Click on Deploy
	
	![](/images/2016-05-29-OracleJVMD/04.png)
	
5.	The progress of the deployment is shown in this screen.

	![](/images/2016-05-29-OracleJVMD/05.png)
	
# Review the JVMD configuration
After installing the JVMD we can review the configuration using the steps described in [2].

1.	Select the option Setup>Middleware Management>Application Performance Management

	![](/images/2016-05-29-OracleJVMD/06.png)
	
2.	Select the JVM Diagnostic Engine called "jammanagerEMGC_JVMDMANAGER1" and click on Configure.

	![](/images/2016-05-29-OracleJVMD/07.png)
	
3.	We can select the different tabs to review the JVMD configuration, the JVMD Pools, etc. as can be seen in this picture.

	![](/images/2016-05-29-OracleJVMD/08.png)
	
	![](/images/2016-05-29-OracleJVMD/08_2.png)
	
# Configuring Diagnostic Agents

With this steps the diagnostic agent will be configured on the weblogic servers within the GCDomain.

1.	Select the option Setup>Middleware Management>Application Performance Management

	![](/images/2016-05-29-OracleJVMD/09.png)
	
2.	Click on Manage Diagnostic Agents.

	![](/images/2016-05-29-OracleJVMD/10.png)
	
3.	In this case there are two domains. The first one is the GCDomain, which was configured automatically and the second one is the soadomain12c that was configured manually after installing the OEM Cloud Control 12C. For this example the soadomain12c will be used. Whit this in mind, mark these checkboxes and click on Next.

	![](/images/2016-05-29-OracleJVMD/11.png)
	
4.	In this page the system requests the host and domain credentials. As my OEM installation was made on the same host (it is not a good idea, but it is only a demonstration) where my SOA domain resides, I can reuse the host credentials. However, in the case of the domain, I have to enter their credentials.
	
	![](/images/2016-05-29-OracleJVMD/12.png)
	
	In the previous page, the check box “Set As Preferred Credentials” is marked in order to use these weblogic credential for future tasks on the SOA domain.
	
5.	Click on Apply.

	![](/images/2016-05-29-OracleJVMD/13.png)
	
	After that, the page is updated. Click on Next.
	
	![](/images/2016-05-29-OracleJVMD/14.png)
	
6.	Keep the default values and click on Apply then click on Next. 

	![](/images/2016-05-29-OracleJVMD/15.png)
	
7.	Click on Deploy.

	![](/images/2016-05-29-OracleJVMD/16.png)
	
8.	The progress of the operation can be monitored.

	![](/images/2016-05-29-OracleJVMD/17.png)
	
	When the status is updated to “Succeeded” the deployment is complete.
	
	![](/images/2016-05-29-OracleJVMD/18.png)
	
	Now we can see JVM pools in the target navigation panel.
	
	![](/images/2016-05-29-OracleJVMD/19.png)

# References list

[1] Oracle (2015) Installing JVM Diagnostics [Online document] Available from: http://docs.oracle.com/cd/E24628_01/install.121/e22624/jvmd_installation.htm#EMBSC213 (Accessed: 17 May 2016)

[2] Oracle (2015) Using JVM Diagnostics [Online document] Available from: https://docs.oracle.com/cd/E28271_01/install.1111/e24215/ad4j_using.htm (Accessed: 17 May 2016)

