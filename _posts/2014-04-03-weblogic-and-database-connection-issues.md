---
layout: post
title: WebLogic and Database connection issues
categories: WebLogic
tags: [weblogic, database, firewall]
author: catoaune
---

Database connection issues could sometimes be hard to solve for a middleware administrator. Below are three scenarios that we have run into, and that are easy to check, now that you know about them:

## Initial connection pool

When you configure a connection pool in WebLogic (from 10.3.6/12.1) you can set Initial Capacity. Initial Capacity is the number of physical connections to create when creating the connection pool in the data source. Depending on your application(s) you might be tempted to set this to a high value, but be aware of at least two issues.

- It takes longer time to start your WebLogic server (but since you rarely restarts the server that might not be an issue)
- If for some reason your WebLogic server can’t reach your database server, your WebLogic server never reaches RUNNING state, but stops at ADMIN state. Of course your database server should be available all the time (in theory), but if both your database server and WebLogic server have been down in the maintenance window, and your WebLogic server boots faster than the database server, you are out of luck.

## Missing test table

You can enable Test Connections on Reserve (Advanced option on the Connection Pool tab). If you do that, you have to provide a Test Table Name (or a SQL, more about that below). The default SQL code used to test a connection is “select count(*) from TestTableName”. So what happens when someone cleans up the database and accidentally deletes the test table. Well, the SQL will fail, and your connection pool will after a while not have any valid connections.

One way to avoid this is to provide a SQL instead of just a table name, to override the default SQL. If you provide SQL SELECT 1 FROM DUAL you should always get a sane reply from the database, as long as it is up and running.

## Your not so friendly neighbourhood firewall

Servers usually expects tcp keepalive to be two hours, which means that a tcp connection could be idle for two hours before it is considered idle, or before a keep alive packet is sent. Unfortunately most firewalls thinks that one hour is more then enough and drops idle tcp connections after 3600 seconds. Even more unfortunately firewalls tends to just drop packets for closed connections, instead of sending a reject message back (for a firewall facing external traffic it makes sense to drop forbidden packets coming from internet, but for internal traffic a reject would have solved many issues). The result is that after a low traffic period, i.e. during a night, weekend or a public holiday, you have a connection pool full of connections that doesn’t work.

The technically easiest solution, but often the hardest one, is to convince the security guys running the firewall to change the tcp keepalive settings to two hours (7200 seconds). If you also are able to get them to reject and not drop packets between your WebLogic and database servers, you are lucky.

The other alternative is to change the tcp keepalive on every server [(http://tldp.org/HOWTO/TCP-Keepalive-HOWTO/usingkeepalive.html)](http://tldp.org/HOWTO/TCP-Keepalive-HOWTO/usingkeepalive.html), and also change your connection string to include ENABLE=BROKEN [(http://docs.oracle.com/cd/B28359_01/network.111/b28317/tnsnames.htm#NETRF431)](http://docs.oracle.com/cd/B28359_01/network.111/b28317/tnsnames.htm#NETRF431)
