---
layout: post
title: Should the SOA Bundle Patches for 12c be applied to OSB?
categories: oracle soa osb
tags: [Oracle, Patching OSB, SOA]
author: jphjulstad
---

# OSB Patching
We have some customers which only use OSB  - and not SOA Suite. When you want to patch - should you look only at WLS and OSB-patches? The answer is no.

One such reason is documented on MOS: Should the SOA Bundle Patches for 11g and 12c be applied to OSB (Doc ID 2102449.1). It states:
In 12c, since OSB services can use JCA technology adapters, there is value in applying the SOA Bundle Patches where fixes to these adapters are included.

The other reason is because JDeveloper has common features in the two products. For example patch  22226040: java.lang.NullPointer for XQuery File ver 1.0 in JDEV 12.2.1 OSB Proj - is one you would like to use for OSB on 12.2.1. The problem is shown in our blog post: [OSB Patch](http://blog.sysco.no/soa/JDev-OSB_Projects-Migrated/). If the patch does not work - remember to do the cleanup-steps mentioned at the end of the blog.

My advise is to create a predefined Patch Search in MOD so you can monitor existing patches. Here are some of my searches.

![](/images/2016-04-02-osb/patch_search.png)

One good thing you can see is the last time you searched. For example for OSB - then WLS, SOA and OSB are relevant. My advise is to order patches so you see tha latest updates first, and that you at least should add the recommended patches.

![](/images/2016-04-02-osb/patch_columns.png)

It is not easy to see which of the patches are relevant only for JDeveloper/Quickstart, and which are only relevant for server.