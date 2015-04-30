---
layout: post
title: Exalogic guest filename ownership
categories: exalogic
tags: [exalogic, ldap, oracle, nfsv4, nfs, linux, nobody]
author: pah
---

Catalogs/files with ownership nobody:nobody on exalogic guests

You have set up your virtual servers on exalogic with NFSv4 mounted
filesystems. After a reboot of the ldap-master-vServer all catalogs and
files previously owned by oracle:oinstall is now showed with the
ownership nobody:nobody. This is not consistent and might happen from
time to time.

It might look like this:

[root@testhost \~]\# **su - oracle**

oracle-@testhost /home/oracle \# **ls -trl /u01**

*total 18*

*drwxrwxr-x 2 nobody nobody 2 Jul 29 13:33 logs*

*drwxr-xr-x 3 nobody nobody 4096 Aug 12 10:51 bootscript*

*drwxrwxrwx 8 nobody nobody 8 Aug 28 12:53 tools*

*-rw-r--r--. 1 nobody nobody 31 Feb 11 06:40 afiedt.buf*

*-rw-r--r--. 1 nobody nobody 7813 Feb 12 08:06 testfile.zip*


Solution:

Troubleshooting Guide for NFSv4 File Lock & Hang Issues On Exalogic
Linux Environments (Doc ID 1492780.1)

Exalogic Elastic Cloud Software Known Issues (Doc ID 1268557.1)

The first one pointed to the last one. It seems like a BUG on earlier
ZFSSA versions, but the solution also worked here.

Restart LDAP Service on ZFSSA

https://storagenodehost:215/

Goto Configuration --\> Services and locate the Data Service named LDAP.
Restart the service. Important to restart and NOT disable/enable. The
last one will not do it.

![](/images/2015-04-30-exalogic-guest-filename-ownership/nobody_nobody_files.png)

Now log in to a new session and check:

[root@testhost \~]\# **su - oracle**
oracle-@testhost /home/oracle \# **ls -trl /u01**

*total 18*

*drwxrwxr-x 2 oracle oinstall 2 Jul 29 13:33 logs*

*drwxr-xr-x 3 oracle oinstall 4096 Aug 12 10:51 bootscript*

*drwxrwxrwx 8 oracle oinstall 8 Aug 28 12:53 tools*

*-rw-r--r--. 1 oracle oinstall 31 Feb 11 06:40 afiedt.buf*

*-rw-r--r--. 1 oracle oinstall 7813 Feb 12 08:06 testfile.zip*
