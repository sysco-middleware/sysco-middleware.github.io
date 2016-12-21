---
layout: post
title: SFTP Transfer in Oracle Service Bus 12c
categories: osb jdeveloper
tags: [Oracle, Oracle Service Bus, JDeveloper, SFTP, SFTP Transfer, OSB 12c]
author: denzza
---

Requirement:
So, you have a need for transferring files via SFTP in OSB 12c to the client’s SFTP endpoint location.
This post can help you in couple of easy steps.

I presume you have access to your weblogic server file system where your integration will be deployed and of course the SFTP host endpoint where the files will be sent to.


> **“This is a work of fiction. Service names, IP’s, business services, endpoints, hostnames and ports are either the products of the author's imagination or used in a fictitious manner. Any resemblance to actual public keys, living or dead server IP’s, or actual hosts is purely coincidental.”**

Solution:
As a first step let’s open the JDeveloper and create/configure everything we need for OSB 12c project part.
Inside the OSB project design view in the External Services lane do a right click for an options menu.
Here we will use SFTP Transfer option and create a new Business service which will be then used for sending messages via Routing activity.
Hover over the 'Insert Transports' and click on the 'SFTP'.

![](/images/2016-12-20-SFTP_Transfer_in_Oracle_Service_Bus_12c/SFTP_Transport.jpg)

Next step is to setup the Business Service for the SFTP. Just follow the next images from the Wizard.

![](/images/2016-12-20-SFTP_Transfer_in_Oracle_Service_Bus_12c/SFTP_BS_Wizard_1.jpg)


![](/images/2016-12-20-SFTP_Transfer_in_Oracle_Service_Bus_12c/SFTP_BS_Wizard_2.jpg)


![](/images/2016-12-20-SFTP_Transfer_in_Oracle_Service_Bus_12c/SFTP_BS_Wizard_3.jpg)

For the SFTP Host name example let’s use: **sftp-endpoint-example-dev.org**

Folder name that will contain the sent messages is: **folderName**

This information will be probably provided by your client/customer.

As a last step we need to configure the business service’s Transport properties.  Open your business service and select the Transport Details (image below).
Add the Service account with the correct credentials that your SFTP is using.
And the most important property to set is the “**Preferred Public Key Algorithm**”, which you can set based on the what algorithm is used for the Public Key for the SSH. Here in this example I have both ‘dss’ and ‘rss’ key algorithms and I will use rss.

*Later on I will show you how you can obtain the Public Key with a simple one liner command.*

![](/images/2016-12-20-SFTP_Transfer_in_Oracle_Service_Bus_12c/SFTPTransportConfiguration.jpg)

Now your business service is ready to be used inside OSB 12c pipeline.

Step two:

Now we have to deal with the ‘known_hosts’ file which should be located on weblogic.
In 12c this is the location where the ‘known_hosts’ file should be located:

**/u01/oracle/config/domains/osbdomain/config/osb/transports/sftp**

The main thing is to find ‘osbdomain/config’ folders on the server file system, and probably you will be missing some of the folders ‘osb/transport/sftp’ which you will then need to create.
In case you are not missing these folders then probably there is ‘known_hosts’ file already created, if not you will need to create it.
Now open the ‘known_hosts’ file and add the ssh Public Key for the rss algorithm including the IP of the server where the messages will be sent.

It has to look something like this:

![](/images/2016-12-20-SFTP_Transfer_in_Oracle_Service_Bus_12c/KnownHostsFile.jpg)

> **Handle the 'known_hosts' file extra carefully, and make sure that file has no CR/LF at the end.
So the file should always be backed up before changing, or might even be added to subversion.**

To obtain the SSH Public Key you need to GET public key (login to the weblogic with putty and run command from the root):

**ssh-keyscan -t rsa,dsa sftp-endpoint-example-dev.org**

As a last step you need to restart the managed servers on weblogic so the changes apply.
Now your Service Bus 12c project is ready to send the messages via SFTP.

[1]: http://www.darkroastedblend.com/2007/01/stars-planets-scale-comparison.html
[2]: http://www.complex.com/pop-culture/2013/04/gallery-babies-using-technology/9
[3]: http://www.thatjeffsmith.com/data-modeling/
[4]: http://docs.oracle.com/cd/E37547_01/tutorials/tut_ide/tut_ide.html
[5]: http://www.quickmeme.com/meme/3rkpgw
