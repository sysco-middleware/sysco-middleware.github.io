---
layout: post
title: Using JMX to Monitor Oracle Weblogic Partitions
categories: Weblogic server monitoring
tags: [Weblogic, Java Management Extensions, JMX, Partitions, Weblogic multi tenancy, Multitenant]
author: raul
---

## Using JMX to Monitor Oracle Weblogic Partitions ##

In this document a monitor is created using JMX. This monitor is made to show the performance of a Data Source, which is deployed on a partition. Thus, MBeans are used not only to discover managed servers, but also to show partitions. In addition, as the main goal is to show the connection pool’s runtime state (do not forget data sources belong to partitions) MBeans are also used to discover the resource groups related to each partition. Furthermore, the program includes a class that generates JDBC connection leaks to demonstrate how to monitor this kind of problems.

You can download the guide using the following link.

[***JMX and partitions***](/files/guides/JMXMonitoring.pdf)