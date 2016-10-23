---
layout: post
title: Using UMS Adapter to send emails with OSB 12c
categories: osb messaging
tags: [OSB, UMS]
author: cliops
---
In this post, we will how to send emails using a UMS adapter in OSB 12c. There we go:

### Configuration in the server side ###

First, we need to verify that we have the UMS adapter and the email driver already working. These configurations are displayed in deployments of weblogic console.

![](/images/2016-10-21-UMS_Adapter_send_email_OSB_12C/UMSWeblogicDeployments.jpeg)

Also, we need to know the configuration for the outbound connection displayed under Configuration->Outbound Connections Pools.

![](/images/2016-10-21-UMS_Adapter_send_email_OSB_12C/UMSAdapterConnectionPools.jpeg)

Now, we have to access to EM and go to usermessagingserver and configure the user messaging driver.

![](/images/2016-10-21-UMS_Adapter_send_email_OSB_12C/UMSEmailServerApp.jpeg)

Click on the edit button to open the configuration page of the driver. There you have to define the sender addresses and SMTP protocols.

![](/images/2016-10-21-UMS_Adapter_send_email_OSB_12C/UMSEmailDriverSettings1.jpeg)

Also, we have to set the outgoing mail settings: mailserver host, port, user and password.
![](/images/2016-10-21-UMS_Adapter_send_email_OSB_12C/UMSEmailDriverSettings3.jpeg)

If we don't want to save the mails then mark the checkbox called AutoDelete.

### Creating UMS adapter in JDeveloper 12c ###

After we set the configurations in the server side, now we can create the application and add UMS adapters.
Create a UMS adapter in the external services section as the image bellow.

![](/images/2016-10-21-UMS_Adapter_send_email_OSB_12C/JdeveloperUMSAdapter1.jpeg)

Then we have to be sure that the connection JNDI Name is eis/ums/UMSAdapterOutbound.

![](/images/2016-10-21-UMS_Adapter_send_email_OSB_12C/JdeveloperUMSAdapter2.jpeg)

In the next step, set the option Outbound Send Notification.

![](/images/2016-10-21-UMS_Adapter_send_email_OSB_12C/JdeveloperUMSAdapter3.jpeg)


In the Step 4 we Set Email option and some additional parameters like Subject, From, To, Cc, Bcc and ReplyTo.

 ![](/images/2016-10-21-UMS_Adapter_send_email_OSB_12C/JdeveloperUMSAdapter4.jpeg)

 Here, if we want to define more than one email then we will get an error message that we only can set one, but for this case we have some workarounds like modifying manually the jca file including more than one email with separate commas. Also we can add more emails in the transport header.

 ![](/images/2016-10-21-UMS_Adapter_send_email_OSB_12C/JdeveloperUMSAdapter5.jpeg)

  Click, next and finish. Then enjoy your UMS adapter to send emails :). I'm checking how to send html DOM objects and some attached emails, so I will create the second part of this post soon!
