---
layout: post
title: Handle locked rows by DB Adapters for distributed Polling technologies in OSB 12c
categories: osb dbpollers
tags: [OSB, DBpollers]
author: cliops
---
In this post, we will learn some ways to handle a database row that is locked by a Databse Adapter with Distributed polling capablity in OSB 12c. There we go:

### Introduction ###

Sometimes, when we work with Database adapters with distributed polling we are struggling to update the same polled row in the respective pipeline because, basically, the polled row is locked with the option "FOR UPDATE SKIP LOCKED" and it will be released when the process finishes or if an error occurs. But, what happen if we want to update the same polled row, let see for example the status description of the process or some error messages. So, I would like to show you some ways to handle these cases.

### 1.- Using JMS resources ###

If we could handle more resources like for example Topics or queues then we would be able to send the polled row to these resources and process them after the poller was released, so it would be useful if we want to release the polled row as soon as we want.

In this case we will create a topic and we will specify the Messaging request as the request of the DB Poller, the column to read and write is called POLLER_READ from 0(READY) to 1(DONE).

 ![](/images/2017-03-15-DBPollers-update-locked-rows-OSB_12C/Image2.jpg)

The next step to do is send the row to JMS topic.

  ![](/images/2017-03-15-DBPollers-update-locked-rows-OSB_12C/Image3.jpg)

In the pipeline of the JMS proxy, the polled row is released so we could use the information of the row to update it self with any information that we want using business services. The challenge here is when the JMS Topic does not work (For example, JMS Server issues) so even when the proxy of the DbPoller could update the POLLER_READ column to 1 this row was not processed correctly in the pipeline of the JMS proxy. For these cases is better to manage another place where to monitor the complete execution of the process flow, for example handle another column like status code to identify if the row was processed in the JMS proxy.


### 2.- Duplicate polled row ###

If we have access to insert in database then we could do the following in the pipeline of the Db poller:

-Poll the row with n retries and physical delete option.

-Insert a row with almost the same content but a different Key(sequence number) or handle a column with the number of retries. Don't forget updating POLLER_READ=1 to the duplicated row.

-Update the duplicated row as we want but don't forget updating the number of retries.

-If there is a handled error then update the duplicated row with POLLER_READ=0 and add a reply with success option to commit the duplicated row and let the row ready to be read one more time.

In the best of the cases it works like the image below:
![](/images/2017-03-15-DBPollers-update-locked-rows-OSB_12C/Image4.jpg)

Otherwise, in the worst of the cases it works like the image below:
![](/images/2017-03-15-DBPollers-update-locked-rows-OSB_12C/Image5.jpg)

This would be posible if we are able to insert data. Also even when we are duplicating a new row at the end of the process always we will have only one of them and we do not have to depend of another resources like the first solution.


### 3.- Exploit the delete operation ###

If we don't have access to external resources (JMS resources) or we are not able to have permissions to write then I would like to propose this solution.

First, create a DbPoller with Delete operation. So we will be able to modify the Poller read query and after read query. Also, I have written about this topic in the following link [Using custom SQL queries for Distributed polling in OSB 12C](http://blog.sysco.no/osb/pollers/Use_custom_sql_Distributed_Polling_OSB_12C/).

Now, what we could do is the following:

-Create an assign component and include the changes that you want. Don't forget poller_read=1 to stop reading the next time.
![](/images/2017-03-15-DBPollers-update-locked-rows-OSB_12C/Image6.jpg)

-Add a replace component to the body after the assign as the image below:
![](/images/2017-03-15-DBPollers-update-locked-rows-OSB_12C/Image9.jpg)

-If there is a handled error then create an assign component with the following content:
![](/images/2017-03-15-DBPollers-update-locked-rows-OSB_12C/Image7.jpg)

-Add a reply component with success option in the handler error. It will help to use the custom after query .

![](/images/2017-03-15-DBPollers-update-locked-rows-OSB_12C/Image8.jpg)

In this case (for error handler) I'm sending POLLER_READ=0 to process again the row so after n retries it will stop. Would be good to handle retries also in database so when the row was polled more than n times it will stop. Otherwise send POLLER_READ=1 and keep the reply component with success.

-Finally open the file xxx-or-mappings.xml with an external text editor and update the querying element as the source below.

```html
<querying>
            <queries>
               <query name="PollClientSelect" xsi:type="read-all-query">
               <call xsi:type="sql-call">
                        <sql>SELECT * FROM CLIENTS WHERE POLLER_READ=0 AND TIMESTAMP &lt;= SYSDATE FOR UPDATE SKIP LOCKED</sql>
                  </call>
                  <reference-class>PollClient.Clients</reference-class>
                  <refresh>true</refresh>
                  <remote-refresh>true</remote-refresh>
                  <lock-mode>lock-no-wait</lock-mode>
                  <container xsi:type="list-container-policy">
                     <collection-type>java.util.Vector</collection-type>
                  </container>
               </query>
            </queries>
  <delete-query xsi:type="delete-object-query">
		<call xsi:type="sql-call">
			<sql>UPDATE CLIENTS SET POLLER_READ=#POLLER_READ, NAME=#NAME,TIMESTAMP=#TIMESTAMP,DESCRIPTION=#DESCRIPTION WHERE ID=#ID</sql>
		</call>
  </delete-query>
</querying>

```
As you can see in the first query we add "FOR UPDATE SKUP LOCKED" to replicate the distributed polling techonology. Also, in the delete query we are updating every field that is coming from the body of the dbpoller.

## Conslusion ##

Basically, depending on what we are going to need and what we are able to do is possible to manage the "locked row" in several situations. If you are not handling distributed polling technologies then is not necessary to apply this solutions because the row is not locked by OSB.

Good luck!
