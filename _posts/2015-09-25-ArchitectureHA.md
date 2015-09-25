---
layout: post
title: Implementing a Weblogic Architecture with High Availability
categories: Weblogic Server High Availability
tags: [HA, AdminServer, Shared Storage, Enterprise Deployment Guide]
author: raul
---

## Implementing a Weblogic Architecture with High Availability ##

### Introduction ###

This document shows a comprehensive process to construct an architecture with high availability that demonstrates how useful is in order to do tasks such as the recovery of the Administration Server in other machine. Even though, the whole server migration of managed servers is not configured in this guide, the architecture is ready to apply this configuration because of the use of virtual IPs and shared storages. In fact the directories that allow this work are also create. For example, the path /u01/oracle/config/domains/incadomain/incacluster is created to store the JMS and TLOG files. This document can be used to learn how to do this configuration and to apply this knowledge to other cases such SOA and OSB.

You can download the guide using the following link.

[High Availability](/files/guides/VirtualEnvironmentV2.1.pdf)