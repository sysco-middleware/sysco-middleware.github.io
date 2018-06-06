---
layout: post
title: Can Coherence be upgraded in a OSB/SOA 11.1.1.7-installation?
categories: coherence
tags: [coherence, soa suite, osb]
author: jphjulstad
---

I am currently doing an install of SOA Suite/OSB 11.1.1.7, and want to apply the latest of patches (WLS, SOA, OSB, JVM). When I looked at the Exalogic Patch Set Update (PSU) I saw that the following patch is part of PSU: p18385410:
Oracle Coherence 3.7.1.12 for Java (COHERENCE 3.7.1.12 FULL DISTRIBUTION).

In the back of my head was also an excellent blog post by Richardo Ferreira: [Enabling WAN Replication for Oracle Service Bus Result Cache](https://blogs.oracle.com/middlewareplace/entry/enabling_wan_replication_on_oracle) that I recommend. One of the pre-reqs there was to upgrade Coherence.

However, when I looked at the following MOS note:  OSB 11g: How to Upgrade Coherence (Doc ID 1605769.1) it showed that:   *Only Coherence 3.7.1.1 is officially supported with OSB 11.1.1.7.0*

I asked MOS if this was the case, and it turned out that OSB is a little more flexible than that. The note has now been updated to:

*Coherence 3.7.x is officially supported with OSB 11.1.1.7*

So if you want to patch Coherence you are now allowed to do so – it contains both new functionality and bug fixes.

The steps are:

* Before applying this patch make a backup copy of the current Coherence install Directory (for instance  $MW_HOME/coherence_3.7_orig). To revert copy the files you backed up back to their original location.
To install expand the .zip file to the directory where the current version of Coherence is located.
* Copy the new coherence.jar to  $MW_HOME/oracle_common/modules/oracle.coherence/coherence.jar
For SOA, note Coherence FAQ for SOA 11g(1471923.1) was also unclear – but is now updated to show that also the 3.7.1.x-versions are supported.

> *3. Is Oracle Coherence 3.7.1.x supported by SOA? SOA 11.1.1.6 comes with Oracle Coherence 3.7.1.1.0, embedded in Weblogic 10.3.6 This version of Coherence has been tested and approved to run with SOA Suite and (x) versions, as released in some PSU’s, are also supported.*
 

So it may be wortwile to patch Coherence if you hit bugs that are fixed in a newer distribution , or that there are enhancements that you would like to make use of.
