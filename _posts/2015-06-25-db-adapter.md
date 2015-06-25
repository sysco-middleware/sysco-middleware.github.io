---
layout: post
title: DB-adapter tips
categories: DB adapter 
tags: [osb, soa suite, db adapter, primary key, rowid]
author: jphjulstad
---
<link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.min.css">
DB adapter tips.

The DB Adapter has some nice features, but sometimes you can get fooled. One of my colleagues experiences this one day. The query returned the same row - just repeated many times.

The reason for this is because the primary key was not defined correctly.

The good thing is that the documentation describes this - and more: [Doc link](http://docs.oracle.com/middleware/1213/adapters/develop-soa-adapters/adptr_db.htm#TKADP1294)

For tables where primary key is defined - this should not be a problem, but in cases where it is not defined or you are querying a view - you would need to specify a primary key yourself. A couple of relevant notes here:

* If you do not provide a valid primary key, then the unique constraint is not guaranteed, and this could result in possible loss of messages at runtime. That is, rows with duplicate primary key values are likely to be lost. 
* You should ensure that you primary key is less than 100 bytes.
* Oracle recommends that you use varchar instead of char for primary key columns

If you do not have a valid primary key - you can use ROWID option for Oracle databases. There are some limitations - for example ROWID can only be used in conjunction with a single table, as relationships between tables require rows to have explicit primary keys. This could happen for instance where a view references multiple tables.

Picking a non-unique column here can lead to operations updating multiple unintended rows, or Selects returning the some rows twice and not others.

Therefore - verify if the primary key you select is a valid one - or you may get fooled.

 