---
layout: post
title: JDeveloper Quickstart 
categories: soa
tags: [Oracle, Weblogic 12.2.1, Weblogic 12.1.3, JDeveloper]
author: jphjulstad
---

# Problems with a space character in Domain
If you have a SOA Quickstart installation on Windows, and your domain directory name contains a space character - you will have a problem starting up the integrated WLS server.

You will see something similar (my directory is named Jon Petter)
```bash
  Error: Could not find or load main class Petter\AppData\Roaming\JDeveloper\system12.2.1.0.42.151011.0031\DefaultDomain
```

To fix this  you can go to the directory mentioned - and change the file: /bin/setStartupEnv.cmd.

From
```bash
  set EXTRA_JAVA_PROPERTIES=%EXTRA_JAVA_PROPERTIES% -Dem.oracle.home=D:\Oracle\Middleware1221\em -DINSTANCE_HOME=C:\Users\Jon Petter\AppData\Roaming\JDeveloper\system12.2.1.0.42.151011.0031\DefaultDomain -Djava.awt.headless=true -Doracle.sysman.util.logging.mode=dual_mode
```

to (adding " before and after in the INSTANCE_HOME):

```bash
set EXTRA_JAVA_PROPERTIES=%EXTRA_JAVA_PROPERTIES% -Dem.oracle.home=D:\Oracle\Middleware1221\em -DINSTANCE_HOME="C:\Users\Jon Petter\AppData\Roaming\JDeveloper\system12.2.1.0.42.151011.0031\DefaultDomain" -Djava.awt.headless=true -Doracle.sysman.util.logging.mode=dual_mode
```
