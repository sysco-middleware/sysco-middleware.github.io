---
layout: post
title: Weblogic Multitenant
categories: Multitenant, platform as a service 
tags: [Oracle, Weblogic, Multitenant]
author: raul
---

## Weblogic 12.2.1 Multi-Tenant ##

With the development of partitions on Weblogic, Oracle has developed an infrastructure that is similar to containers, which takes advantage of the Weblogic server’s capacities such as clustering, transaction management and security [1].

These are the advantages of using Weblogic Server Multitenant [1]:

Time to market is improved.

1.	The complexity of moving workload to the cloud and from the cloud is reduced.
2.	It is possible to convert monolithic applications to smaller services.
3.	It allows up to 3x hardware consolidation.
4.	Reduction of OPEX by up to 25% 

Since Weblogic Multitenant is based on the concept of partitions or micro containers. It is important to remark that these partitions allow the portability of applications reducing the time to market and allowing the movement to the cloud or vice versa. 

Multi-tenant allows group applications that are scattered through several domains, which helps to optimize the use of hardware, making possible the reduction of OPEX.

In addition, a partition does not have any Operating System or JVM component. Applications and configuration artefacts compose partitions or micro containers and each one of these micro containers could use a managed server or a cluster. 

In the following diagram, the topology shows two partitions deployed on the same cluster, which allow them sharing the JVMs that are part of that cluster.

![](/images/2016-01-12-Multitenant/01.png)

With this in mind, in this post, I will show you how to reach the topology described based on partitions. I have created a domain with a cluster and I have an Oracle Pluggable Database available so now these are the additional elements created in this post:

1.	Virtual targets. According to [2] a virtual target is the target used by a resource group at the domain level and partition level. Virtual targets are targeted to managed servers or clusters and they define access points to resources.
Virtual targets give a separate HTTP per each server as in the case of virtual hosts in Weblogic Server [2].
Since virtual targets set the access to resources and resources are group by resource groups, these require one or more virtual targets. When a resource group has a global scope (related to the domain) it is possible to select any virtual target that is not assigned to a partition. On the other hand, when a resource group is assigned to a partition, this can use only available virtual targets in the partition [2].

2.	Resource groups. We can define it with an example, many times when we need two environments like development and testing, we should create two domains in order to separate applications and resources such as data sources, JMS servers, etc. With the introduction of Weblogic Multi-Tenant, you can group applications and resources to create a resource group with the elements needed by each environment. In addition, we should create partitions (one per each environment) to target one or more resource groups on these partitions. This configuration can be seen in the following picture.

![](/images/2016-01-12-Multitenant/02.png)

Since this configuration consolidates several domains into one, this case is a platform-as-a-service scenario because each partition is a Weblogic platform to deploy applications and resources [2].

In addition [2] states that resource groups give coherence to application suits that before were scattered in a domain. 

Resources groups can be created at the domain level or at the domain partition level. A resource group created at the domain level has a global scope and cannot be used by any partition. On the other hand, a resource group created at the partition level has an scope that only covers that partition, it means that applications at this level are not available at the domain level or for other partitions [2].

1.	Partitions. According to [2] a partition is an administrative and runtime unit that is equivalent to a portion of a domain, which is used to run applications and their resources. Oracle recommends that you should not create more than 10 partitions per domain [2]. However, this number is up to you, but you have to consider the particular conditions of your deployments.

2.	Security realms. I will create a security realm per partition in order to manage the security independently.

3.	Partition users. The security realms created in the previous step will be used to define administrative users in charge of the administration of each partition.

### Create virtual targets ###

Click on Virtual Targets under Environment

![](/images/2016-01-12-Multitenant/03.png)

Click on New

![](/images/2016-01-12-Multitenant/04.png)

In this screen you have to define the name of the Virtual Target, the target used by the virtual target, the hostnames configured to attend requests and the URI prefix to differentiate the development environment from the testing environment. Click on Ok

![](/images/2016-01-12-Multitenant/05.png)

Do the same for the testing environment.

![](/images/2016-01-12-Multitenant/06.png)

### Create security realm for the partitions ###

Click on security realms

![](/images/2016-01-12-Multitenant/07.png)

Click on New

![](/images/2016-01-12-Multitenant/08.png)

Enter the new realm name, mark the two checkboxes and click on Ok.

![](/images/2016-01-12-Multitenant/09.png)

Do the same for testing

![](/images/2016-01-12-Multitenant/10.png)

### Create users for the partitions ###

Click on security realms

![](/images/2016-01-12-Multitenant/11.png)

Click on the new realm created for the development environment

![](/images/2016-01-12-Multitenant/12.png)

Click on the users and groups tab

![](/images/2016-01-12-Multitenant/13.png)

Click on new.

![](/images/2016-01-12-Multitenant/14.png)

Fill the data remarked and click on Ok.

![](/images/2016-01-12-Multitenant/15.png)

Do the same on the realm that belongs to the testing environment

![](/images/2016-01-12-Multitenant/16.png)

### Give administrative privileges to the new users ###

Click on each one of the new users

![](/images/2016-01-12-Multitenant/17.png)

Click on groups tab

![](/images/2016-01-12-Multitenant/18.png)

Select the Administrators group and click on save.

![](/images/2016-01-12-Multitenant/19.png)

Do the same for the user that belongs to the testing environment.



### Create partitions

Click on domain partitions

![](/images/2016-01-12-Multitenant/20.png)

Click on new

![](/images/2016-01-12-Multitenant/21.png)

Set a name for the partition, unmark the checkbox because we will create the resource group and click on next.

![](/images/2016-01-12-Multitenant/22.png)

Select the virtual target that belongs to the development environment and click on next.

![](/images/2016-01-12-Multitenant/23.png)

Select the virtual target and the security realm created for this partition and click on finish

![](/images/2016-01-12-Multitenant/24.png)

Do the same for the testing environment

![](/images/2016-01-12-Multitenant/25.png)

At the end, we will have two partitions

![](/images/2016-01-12-Multitenant/26.png)

### Create resource groups ###

It is important to remark that we will create resource groups at the partition level so you have the follow these steps.

Click on domain partitions

![](/images/2016-01-12-Multitenant/27.png)

Click on the partition you want to configure, in this case the development environment partition

![](/images/2016-01-12-Multitenant/28.png)

Click on resource groups

![](/images/2016-01-12-Multitenant/29.png)

Click on new

![](/images/2016-01-12-Multitenant/30.png)

Set the name for the resource group, unmark the checkbox highlighted, select the virtual target available for this partition and click on Ok.

![](/images/2016-01-12-Multitenant/31.png)

This is the result

![](/images/2016-01-12-Multitenant/32.png)

Do the same for the test environment

![](/images/2016-01-12-Multitenant/33.png)

### Configure the application

In this post, we will use a demo application called WeblogicMT.war; this is a web application that just executes a query against a pluggable database and shows the results on a dynamic page. In addition, it allows adding and deleting registers on a table called APEX_040200.WWV_DEMO_EMP. The following diagram describes the application architecture.

![](/images/2016-01-12-Multitenant/33.1.png)

Therefore, we need to create the data sources and deploy the applications per each partition with the following steps.

### Create the data source

Click on data sources

![](/images/2016-01-12-Multitenant/34.png)

Click on new generic data source

![](/images/2016-01-12-Multitenant/35.png)

Set a name for the data source, select the scope in this case the development resource group is selected, set the JNDI name, select the kind of database and click on Next.

![](/images/2016-01-12-Multitenant/36.png)

Select the driver remarked and click on Next

![](/images/2016-01-12-Multitenant/37.png)

Click on Next.

![](/images/2016-01-12-Multitenant/38.png)

Fill the connection parameter for pdb1.sysco.no (the pluggable database for the development environment) and click on Next.

![](/images/2016-01-12-Multitenant/39.png)

Test the configuration and click on Next

![](/images/2016-01-12-Multitenant/40.png)

Click on finish

![](/images/2016-01-12-Multitenant/41.png)

Do the same for the test environment. However, we have to use the other pluggable database pdb2.sysco.no and other resource group PartitionResourceGroupTest.

![](/images/2016-01-12-Multitenant/42.png)

Finally, we have these data sources.

![](/images/2016-01-12-Multitenant/43.png)

### Deploy the application

Click on deployments

![](/images/2016-01-12-Multitenant/44.png)

Click on Install

![](/images/2016-01-12-Multitenant/45.png)

Select the application and click on next

![](/images/2016-01-12-Multitenant/46.png)

Select the resource group and click on next

![](/images/2016-01-12-Multitenant/47.png)

Click on next

![](/images/2016-01-12-Multitenant/48.png)

Click on Finish

![](/images/2016-01-12-Multitenant/49.png)

Do the same for the development application, but using a different scope. Finally, we will have:

![](/images/2016-01-12-Multitenant/50.png)

### Testing the environments

### Development

In order to test the development environment, I used this URL: 

http://ms01vhost122.sysco.no:9003/appdesa/WeblogicMT

I clicked on “Edit”

![](/images/2016-01-12-Multitenant/51.png)

I changed the name as can be seen

![](/images/2016-01-12-Multitenant/52.png)

After saving the register I run a query against pdb1.sysco.no, the pluggable database that belongs to the development environment to verify the change. 

![](/images/2016-01-12-Multitenant/53.png)

### Testing

In this case, this address was used: 

http://ms01vhost122.sysco.no:9003/apptest/WeblogicMT

I made a similar change that was verified on the pdb2.sysco.no database, the pluggable database that belongs to the testing environment.

![](/images/2016-01-12-Multitenant/54.png)

### What about security?

Two security realms with a user per each realm were created. We have this information.

![](/images/2016-01-12-Multitenant/54.1.png)

Now we can use these users to connect to each partition

Connecting to the development environment.

![](/images/2016-01-12-Multitenant/55.png)

Connecting to the testing environment.

![](/images/2016-01-12-Multitenant/56.png)

### References

[1] Oracle (2015) Oracle WebLogic Server Multitenant: The World’s First Cloud-Native Enterprise Java Platform [Online document] Available from: http://www.oracle.com/us/products/middleware/cloud-app-foundation/weblogic/weblogic-server-multitenant-ds-2742664.pdf (Accessed: 06 January 2016)

[2] Oracle (2015) Configuring Resource Groups [Online document] Available from: https://docs.oracle.com/middleware/1221/wls/WLSMT/config_resource_grp.htm#WLSMT1213 (Accessed: 06 January 2016)
