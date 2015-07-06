---
layout: post
title: JVM memory leak 
categories: Memory leak 
tags: [Java, JVM, garbage collector, memory, leak]
author: raul
---

## Using Oracle Cloud Control 12C to Analyse a Memory Leak Problem  ##

### 1.	Introduction ###
The aim of this post is to use the Oracle Cloud Control 12C to find the root cause of a memory leak problem. One of the main advantages of Java is the use of a garbage collector that allows programmers to forget about the memory management. That is not the case of languages such as C++ where you have to allocate and free memory since your code. In my experience in a big company, I saw memory leak problems several times causing the unavailability or the bad performance of many applications.

In addition, a memory leak belongs to a class of situations called software aging problems. Software aging defines the loss of performance over the time because of the gradual accumulation of little problems. Other term within this kind of problem is the rejuvenation process. For example, rejuvenation happens when the system has to be restarted in order to free the memory accumulated by a Java Virtual Machine (JVM) process after a period of time (COTRONEO, D, et al., 2015).

However, there is a question about this problem on JVM. Taking into account that JVM has a garbage collector, how does JVM accumulate memory over the time? The answer is in relationship with the way that the garbage collector use to free memory, the codification on Java and even the use of some parameter within the configuration. 

JVM uses a generational model to manage the memory assigned to itself so the following lines will explain the dynamic of the JVM memory. After this, Oracle Cloud Control 12 C will be used to diagnostic the cause of the problem.

### 2.	Java Memory ###
The java memory model used here is the Oracle Hot Spot model that is composed by the sectors that can be seen in the following picture.

![](/images/2015-07-03-memory-leak/01.png)

Following each one of this components will be described.

### 2.1.	Heap ###
This is the memory place where objects (instances of a class) are saved. With this in mind, taking into account that Java is a garbage collector language, the minimum and maximum thresholds have to be tuned carefully in order to optimize the collection of objects without any reference. It means objects that are not used by any object (Oracle, n.d.). In addition, we have to analyse the sizing of our system to avoid the CPU consumption caused by garbage collection execution and the consequently crash of the system due to the lack of enough memory. How can I set the size of the heap? You have to use these parameters:

* Heap minimum size: 	-Xms
* Heap maximum size: 	-Xmx 

Furthermore the heap is divided in two places the young generation and the old generation.

### 2.1.1.	Young generation ###
This is the place where objects generated after the execution of the sentence new() are saved. This memory space is divided in two part the new generation or Eden and the survivor space (Oracle, n.d.).

### 2.1.1.1.	Eden or new generation ###
When an object has just created using the sentence new() it is saved here. When this place is full, a minor garbage collection is executed to eliminate objects without any reference. Referenced objects are moved to one of the survivor spaces. In addition, the age of referenced objects is incremented by one (Oracle, n.d.).

### 2.1.1.2.	Survivor space ###
This place keeps referenced objects, which are moved from the Eden during the minor collection execution. There are two survivor spaces S0 and S1 (Oracle, n.d.).

### 2.1.2.	Old generation ###
This segment of memory contains older objects that are still referenced. This objects are collected from the young generation when the execution of a minor garbage collection found objects that have passed the age threshold. This threshold is defined by the parameter: MaxTenuringThreshold (Oracle, n.d.).

### 2.1.3.	A little description about the garbage collector work on the heap. ###
Let us suppose we have a JVM whose MaxTenuringThreshold was set on 4 and a class called RM.java, which will be instanced several times (Oracle, n.d.).

i.  After creating several objects from the class RM. Java using the instruction RM rm1=new RM(), they will be stored in the Eden within the young generation space (Oracle, n.d.).

ii.	After some time the Eden will be filled by the creation of objects and then a minor garbage collection will be triggered. This minor collection will eliminate objects without reference and will move the object, which are being used (they have references) into the survivor segment, let us say S0 (Oracle, n.d.).

iii.	If the Eden space is filled again because of the use, the minor collection will be executed. In this case the survivor objects (with references, in use) will be moved into the survivor space called S1. In addition, objects with references from the survivor space S0 will be moved into the S0 segment increasing their age by 1. Eventually, all the survivor objects are moved to the S1 and both the Eden and the S1 will be empty (Oracle, n.d.).

iv.	When a minor collection is executed again, the previous process runs again, but this time survivor objects are moved from the Eden into the S0 and from the S1 into the S0. In addition the age of survivor objects is increased by 1 (Oracle, n.d.).

v.	Over the time some will reach the age of 4 (MaxTenuringThreshold=4) and then these object will be moved into the old generation space (Oracle, n.d.).

vi.	When the old generation space is full, a major garbage collection is executed over this segment (Oracle, n.d.).

### 2.2.	Non heap space
This space contains the JVM native code and the classes’ metadata. These classes are loaded by the applications deployed on the server. It is important to remark that this segment stores metadata, it does not stores objects (Oracle, n.d.). For example, native code used to improve the Just in Time compiler uses this space. In this guide the memory leaks of this spaced will not be analysed.

### 3.	Memory leak analysis
Now we have some concepts about java memory, we can analyse a memory leak problem using Oracle Cloud Control 12c.

### 3.1.	Configuring the Java Pool
From the setup menu select Middleware Management > Application Performance Management option

![](/images/2015-07-03-memory-leak/02.png)

Select the JVMD engine and click on Configure.

![](/images/2015-07-03-memory-leak/03.png)

In the JVMs and Pools tab select the pool related to the managed server that contains our application and click on Configure.

![](/images/2015-07-03-memory-leak/04.png)

In this screen modify the path where the heap dump will be saved and click on Save.

![](/images/2015-07-03-memory-leak/05.png)

### 3.2.	Using Grinder to stress the application.
In this case the application has a bug that will generate a memory leak scenario. In order to accelerate the process, Grinder is used to create thousands of requests.

This is the heap state before executing the test

![](/images/2015-07-03-memory-leak/06.png)

It is important to realize that before the test, the number of major garbage collections is zero.

![](/images/2015-07-03-memory-leak/07.png)

I generated a heap dump report before stressing the application server. These are the steps.

1.	Select the option “Heap Snapshots and Class Histograms”

![](/images/2015-07-03-memory-leak/08.png)

2.	Click on “Create”

![](/images/2015-07-03-memory-leak/09.png)

3.	Select “JVMD Format (txt)” and “All”

![](/images/2015-07-03-memory-leak/10.png)

4.	Click on “Submit”

![](/images/2015-07-03-memory-leak/11.png)

5.	This is the result of the job, which executes the heap dump.

![](/images/2015-07-03-memory-leak/12.png)

6.	In this table, it is possible to see the new heap loaded within the system

![](/images/2015-07-03-memory-leak/13.png)

Execution of Grinder, now is time to stress the application using Grinder to simulate several sessions.

Start the console

![](/images/2015-07-03-memory-leak/14.png)

Start the agent

![](/images/2015-07-03-memory-leak/15.png)

Start the applications requests

![](/images/2015-07-03-memory-leak/16.png)

After some minutes the pressure can be felt in the number of Major GC that now is equal to 9.

![](/images/2015-07-03-memory-leak/17.png)

Another interesting indicator is that even though 9 major garbage collections were executed, the “JVM heap used after GC” is increasing. 

![](/images/2015-07-03-memory-leak/18.png)

### 3.3.	Getting the memory leak report
In order to get the memory leak report to find the root cause of this problem, you have the follow these steps.

1.	Right click on the sever affected and then select Diagnostics > Heap Snapshots and Class Histograms

![](/images/2015-07-03-memory-leak/19.png)

2.	Click on Create

![](/images/2015-07-03-memory-leak/20.png)

3.	Select the radio button “JVMD Format (txt)” and the check box “All”

![](/images/2015-07-03-memory-leak/21.png)

4.	Click on Submit

![](/images/2015-07-03-memory-leak/22.png)

5.	After some minutes the job in charge of the heap processing will finish showing something like this.

![](/images/2015-07-03-memory-leak/23.png)

6.	In order to get the report, you have to select one the heap generated by the previous process and then click on Detail.

![](/images/2015-07-03-memory-leak/24.png)

7.	Click on the tab Memory Leak Report

![](/images/2015-07-03-memory-leak/25.png)

8.	According to this page, we have a memory leak problem and two candidates

![](/images/2015-07-03-memory-leak/26.png)

9.	So we have to get into details to see who is causing the problem

In the first case we have the class weblogic.servlet.internal.HttpServer.SessionLogin

![](/images/2015-07-03-memory-leak/27.png)

While in the second case we have the class weblogic.servlet.internal.session.MemorySessionContext

![](/images/2015-07-03-memory-leak/28.png)

With this in mind, it seems to be the problem is related to the management of the application’s web sessions. Thus, we have to review the web.xml file to get some details about this. This is the file.

``` xml

<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
    <session-config>
        <session-timeout>
            -1
        </session-timeout>
    </session-config>
    <servlet>
      <servlet-name>DeadLockServlet</servlet-name>
      <servlet-class>DeadLockServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>DeadLockServlet</servlet-name>
        <url-pattern>/deadlock</url-pattern>
    </servlet-mapping>
</web-app>
```

After reviewing the file we can see there is a problem with the timeout. For some reasons it was set in “-1” it means web session will live forever and then garbage collector could not free the JVM heap. This case happened in a big company, but the difference was that there were several web applications. However, I think that this guide could be useful to learn about how to find the root cause of a memory leak problem using the Oracle Cloud Control, at that time I had to use the Eclipse Memory Analyser Tool, but it is another history. Last but not least, it is advisable the use of methodologies to find the root cause of a problem in order to avoid scheduled restarts to rejuvenate the application server.