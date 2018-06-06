---
layout: post
title: DB Adapter - Native sequencing, SequencePreallocationSize 
categories: DB adapter 
tags: [osb, soa suite, db adapter, SequencePreallocationSize ]
author: jphjulstad
---
<link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.min.css">
Here are some DB adapter tips regarding use of native sequencing.

In cases where you want to use the database adapter for inserts, you may want to use native sequencing to populate a primary key.

One problem that you can get into is documented in the [orafmw blog]  (https://orafmw.wordpress.com/2011/01/28/sequence-increment-and-pre-allocation-size/)

“The sequence named [ORDER_SEQ] is setup incorrectly. Its increment does not match its pre-allocation size.”

This issue arises because DbAdapter by default sets pre-allocation size as 50. This should match with Database Sequence’s increment size. I usually go in and set this one to 1 as shown in the blog.

One reason why should keep a larger number here is for performance. The [Oracle Fusion Middleware Performance and Tuning Guide, Oracle Adapters Performance Tuning] (http://docs.oracle.com/cd/E14571_01/core.1111/e10108/adapters.htm#ASPER327) chapter 15.3.1 suggests 

* Use Native Sequencing

If you are using the XSL functions to assign primary keys to records, consider using the built-in native sequencing support in the adapter. Sequencing support obtains and caches 50 keys at a time by default. Caching improves performance by reducing the number of round trips. The chunk size can be controlled incrementally by modifying the sequencePreallocationSize connector property.

Other interesting performance improvements for DB adapter relevant for Exalogic - I will look more into that later. Here is the reference to doc [Oracle Fusion Middleware User's Guide for Technology Adapters, Oracle JCA Adapter for Database] (http://docs.oracle.com/cd/E23943_01/integration.1111/e10231/adptr_db.htm#TKADP2434) chapter 9.8.3