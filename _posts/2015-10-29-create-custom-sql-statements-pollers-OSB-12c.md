---
layout: post
title: Nice trick to create custom sql statements in polling DbAdapters with OSB 12c
categories: OSB
tags: [OSB,Pollers]
author: cliops
---
Now we will learn how to change the sql statement for Database pollers using OSB 12c.
There we go:

## Introduction ##

OSB allows us to create DbAdapters and select a polling functionality, but there are some restrictions if we want to write a custom sql statement.   
There is a nice way to set a pure sql statement for polling database rows and after reading operation. I will show you how to handle it.   

## Create poller ##

First, we have to create a dbAdapter on proxy services and select "Poll for New or Changed Records in a Table".   

- ![](/images/2015-10-29-create-custom-sql-statements-pollers-OSB-12c/createPoller.jpg)

In the "After Read" step select "Delete the Row(s) that were Read" option like the image below:   

- ![](/images/2015-10-29-create-custom-sql-statements-pollers-OSB-12c/createDeletePoller.jpg)

If you are using clusters then select Distributed Polling (FOR UPDATE SKIP LOCKED) . This will help to avoid record locking issues.
 
- ![](/images/2015-10-29-create-custom-sql-statements-pollers-OSB-12c/distributedPolling.jpg)

## Edit poller ##

Then it will create 6 files and one of them finishes in -or-mapping.xml, this file is protected, so you have to open in another text editor.
Inside of this file you can set the pure sql for the poller and after reading operations.

To set poller sql statement:   

```xml 
 <querying>
	<queries>
		 <query name="TestPoller" xsi:type="read-all-query">
			<call xsi:type="sql-call">
				<sql>Pure SQL statement</sql>
			</call>
		</query>
	</queries>
 </querying>
``` 

To set pure sql after read   

```xml
	.
	.
	.
	</queries>
	<delete-query xsi:type="delete-object-query">
		<call xsi:type="sql-call">
			<sql>After read SQL statement</sql>
		</call>
	</delete-query>
</querying>
``` 

## Conclusion ##

Basicaly, we can execute any pure sql statement selecting this option in the DbAdapter.   
When we select this option we will not delete some row if we add these codes to overwrite the delete function with another sql statement.





