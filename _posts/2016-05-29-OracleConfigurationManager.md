---
layout: post
title: Using Oracle Configuration Manager
categories: WebLogic
tags: [Oracle, WebLogic, Java, JVM]
author: raul
---

# Introduction
In this little guide an Oracle Fusion Middleware 11G installation is used to demonstrate how to configure Oracle Configuration Manager 12 (OCM). During my experience as a System Administrator, I have seen with surprise that many customers do not use this system because they do not have enough time to configure it or because of the lack of information about it. However, it is important to remark that OCM gives customers the following advantages (Oracle, 2016).

“Some of the benefits of using Oracle Configuration Manager are as follows:

•	Reduces time for resolution of support issues

•	Provides pro-active problem avoidance

•	Improves access to best practices and the Oracle knowledge base

•	Improves understanding of customer's business needs and provides consistent responses and services”

# The installation used
The Oracle Fusion Middleware installation used in this guide is described as follows.

![](/images/2016-05-29-OracleConfigurationManager/01.png)

# Configuring Oracle Configuration Manager
1.	Connect to the machine where OCM will be installed. In this case is: weblogic01.sysco.no

2.	Set the domain environment

	cd /u01/oracle/config/domains/skageraktest/bin
	
	. ./setDomainEnv.sh

	It is important to remark you have to use ". ./setDomainEnv.sh", otherwise values are not set on variables.

3.	Set the Oracle home
	
	export ORACLE_HOME=/u01/oracle/products/fm11117/utils/

4.	Change folder to 

	/u01/oracle/products/fm11117/utils/ccr/bin
	
5.	Execute this command

	./setupCCR

6.	Follow these instructions

	![](/images/2016-05-29-OracleConfigurationManager/02.png)

	![](/images/2016-05-29-OracleConfigurationManager/03.png)
	
7.	Start the Administration Server to generate the file domainlocation.properties whose path is:

	/u01/oracle/products/fm11117/utils/ccr

8.	Execute a manual collection

	cd /u01/oracle/products/fm11117/utils/ccr/bin/
	
	./emCCR collect
	
	![](/images/2016-05-29-OracleConfigurationManager/04.png)
	
9.	Review the configuration on My Oracle Support > Tab Systems. As can be seen in the following picture all my configuration is available on Oracle Support that represents and advantage to minimize the time needed to solve cases or to open service requests.

	![](/images/2016-05-29-OracleConfigurationManager/05.png)
	
# Testing the OCM configuration
In this case one of my favourites capacities will be tested, the detection of configuration changes on the environment. With this in mind, a data source is modified to review whether OCM detects the change or not.

1.	Modify the data source called EDNDataSource to have a maximum of 60 connections instead of 20

	![](/images/2016-05-29-OracleConfigurationManager/06.png)
	
2.	Execute a manual collection with the following command.

	cd /u01/oracle/products/fm11117/utils/ccr/bin/
	
	./emCCR collect

3.	Go to Oracle Support to review the history of changes. As can be seen in the following picture the change is detected by OCM.

	![](/images/2016-05-29-OracleConfigurationManager/07.png)
	
Why is the information shown in the previous picture important? This is relevant because between other things it allows system administrators to investigate the root cause of problems without losing time on exhaustive searching that is critical for severity one incidents.

# References list

Oracle (2016) Configuration Manager Installation and Administration Guide [Online document] Available from:  http://docs.oracle.com/cd/E49269_01/doc.12/e48361/ch1_introduction.htm#CCRIA109 (Accessed: 02 May 2016)
