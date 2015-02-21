---
layout: post
title: Using RDAs health check
categories: Health_check
tags: [health check, rda]
author: jph
---

How can you know that your SOA installation is healthy? One thing is patching – you must read notes on MOS or search MOS for suitable patches (or let EM12c help you). One other potential problem is keeping the configuration according to Best Practices. If you have filed a Service Request(SR), you know that the first thing Support will tell you is to send them an RDA. This blog will tell why this report is not only useful for Oracle Support, but also for you.

## Setup ##

The first thing you should do is to keep RDA up to date – the latest Version is 8.0.3. A useful bulletin on MOS is: Resolve Problems Faster! Use Remote Diagnostic Agent ( RDA ) – Fusion Middleware and WebLogic Server (Doc ID [1498376.1](https://support.oracle.com/epmos/faces/DocumentDisplay?id=1498376.1)).

![RDA Bulletin on MOS](/images/rda_bulletin-610x489.png)

The good news is that RDA now can be updated via OPatch. RDA is a tool for many products, so you should configure accourding to your installation. This bulletin is interactive, so you can select product and the description will update. For a SOA install there may be many relevant Products: SOA, OSB, BAM, WLS, JVM.

![RDA SOA Config](/images/RDA_SOAConfig-610x489.png)

CAUTION : Do NOT run “rda.sh” or “rda.cmd” without parameters unless you have created a result set configuration! If RDA finds no result set configuration file ([result set name].cfg) then running the RDA command script with no parameters will launch the result set configuration process for all data collection modules.

## Heath checks ##

You can find information about Heath checks here:

![Health Check in Bulletin](/images/rda_bulletin_health_check-610x66.png)

Remote Diagnostic Agent also offers a health check tool – known as the HCVE Rule Engine. A full description of the HCVE Rule Engine can be found in

[Note: 250262.1](https://support.oracle.com/epmos/faces/DocumentDisplay?id=250262.1) RDA – Health Check / Validation Engine Guide

To launch the rule engine, run the command

```bash
Unix:
 
    rda.sh -dT hcve
 
MS Windows:

    rda.cmd -dT hcve
```

The command will list the available rule sets for the host platform. For example, if you run the Rule Engine against Linux x86 you would see something like:

```bash
[test]$ rda.sh -dT hcve 
 Processing HCVE tests ... 
 Available Pre-Installation Rule Sets: 
 1. Oracle Database 10g R1 (10.1.0) Preinstall (Linux) 
 2. Oracle Database 10g R2 (10.2.0) Preinstall (Linux) 
 3. Oracle Database 11g R1 (11.1) Preinstall (Linux) 
 4. Oracle Database 11g R2 (11.2.0) Preinstall (Linux) 
 5. Oracle Application Server 10g (9.0.4) Preinstall (Linux) 
 6. Oracle Application Server 10g R2 (10.1.2) Preinstall (Linux) 
 7. Oracle Application Server 10g R3 (10.1.3) Preinstall (Linux) 
 8. Oracle Fusion Middleware 11g R1 (11.1.1) Preinstall (Linux) 
 9. Oracle Portal Preinstall (Generic) 
 10. Oracle Identity Management 10g (10.1.4) Preinstall (Linux) 
 11. Oracle Business Intelligence Enterprise Edition 11.1.1 Preinstall (Generic) 
 12. Oracle E-Business Suite Release 11i (11.5.10) Preinstall (Linux x86 and x86_64) 
 13. Oracle E-Business Suite Release 12 (12.1.1) Preinstall (Linux x86 and x86_64) 
 14. Oracle Enterprise Performance Management 11.1.2 Server Preinstall(Generic) 
 Available Post-Installation Rule Sets: 
 15. Oracle WebLogic Server 10 Post Installation (generic) 
 16. Oracle WebLogic Server 12 Post Installation (generic) 
 17. Oracle Portal Post Installation (generic) 
 18. Oracle SOA 11g (11.1.1) Post Installation 
 19. Oracle OSB 11gR1 (11.1) Post Installation 
 20. RAC 10G DB and OS Best Practices (Linux) 
 21. Data Guard Post Installation (Generic) 
 Enter the HCVE rule set number 
 Hit 'Return' to accept the default (1)
```

Here are some sample checks:
![Sample posts checks](/images/rda_post_checks-610x213.png)


A full listing of the current health checks can be found [here](https://support.oracle.com/epmos/faces/DocumentDisplay?id=250262.1#rulesets). Information and instructions regards SOA health checks can be found in:  [Note: 1571554.2](https://support.oracle.com/epmos/faces/DocumentDisplay?id=1571554.2) How to Run Remote Diagnostic Agent ( RDA ) Against SOA Products.

The Health check contains two types of items:

* Facts for extracting information
* Rules for checking compliance

Here is a sample fact:
![RDA Fact](/images/rda_fact-610x47.png)

Here is a sample rule:
![RDA Rule](/images/rda_rule-610x304.png)

This means that configuration problems that have been identified and described on MOS now can be checked automaitcally in your installation. Here is a sample from a report:
![RDA Report](/images/rda_report-610x343.png)

When is the appropriate time to run RDA Health checks?

* After installation
* After configuration changes
* When a new version of RDA is installed


