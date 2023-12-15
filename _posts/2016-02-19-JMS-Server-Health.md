---
layout: post
title: JMS Server Health Check 12.2.1
categories: soa
tags: [Oracle, Weblogic 12.2.1, JMS]
author: jphjulstad
keep: no
---

# JMS Server health check not showing in WLS 12.2.1
One of the problems we saw on a recent OSB 12.2.1-installation was that the JMS Server health check did not display. In EM console we could see the status show up correctly.

Fortunately Oracle has very recently released a patch for this, and we could today verify that it solved our problem. Here are the patch details: Patch 21830665: HEALTH CHECK RESULT OF JMS SERVERS ARE LEFT BLANK.

At the same time we applied the Patch 22331568 which is the latest Recommended Bundle: WLS PATCH SET UPDATE 12.2.1.0.160119.

Oracle have released some patches on WLS, OSB and SOA the last week - so take a look at MOS.
