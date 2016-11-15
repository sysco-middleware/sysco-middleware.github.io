---
layout: post
title: Using custom SQL queries for Distributed polling in OSB 12C
categories: osb pollers
tags: [OSB, Pollers]
author: cliops
---
In this post, we will learn how to use customized queries for Distributed polling in OSB 12C, there we go:

### SOLUTION  ###

First, we need to know how to create a Db poller in OSB 12C using JDeveloper. Actually, I made a previous post about how to acomplish this step in the following link:
[create custom sql statements in polling DbAdapters with OSB 12c](http://blog.sysco.no/osb/create-custom-sql-statements-pollers-OSB-12c/)

So basically, after we created a Database poller with the option of physical delete (first option) then we will modify the xxx-mappings.xml file with a text editor because this file is protected in Jdeveloper.

```xml
  <querying>
            <queries>
               <query name="customerPollerSelect" xsi:type="read-all-query">
                 <!--QUERY FOR POLLING-->
                  <call xsi:type="sql-call">
                        <sql>SELECT * FROM CLIENTS WHERE POLLER_READ=0 AND TIMESTAMP &lt;= SYSDATE FOR UPDATE SKIP LOCKED</sql>
                  </call>
                  <reference-class>customerPoller.Clients</reference-class>
                  <lock-mode>lock-no-wait</lock-mode>
                  <container xsi:type="list-container-policy">
                     <collection-type>java.util.Vector</collection-type>
                  </container>
               </query>
            </queries>
            <delete-query xsi:type="delete-object-query">
		<call xsi:type="sql-call">
      <!--QUERY FOR UPDATING AFTER POLLING-->
			<sql>UPDATE CLIENTS SET POLLER_READ=1 WHERE ID=#ID</sql>
		</call>
	</delete-query>
  </querying>
```

In this piece of code, I'm specifying the query for polling and query for updating after polling. Here there is one important thing when we are handling distributed polling, I added "FOR UPDATE SKIP LOCKED". This means that one managed server will process a database row and it won't be processed by another manager server that is polling the same table.
