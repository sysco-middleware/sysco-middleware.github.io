---
layout: post
title: Oracle VM in VirtualBox
categories: OracleVM
tags: [oracle vm]
author: catoaune
keep: yes
---

Oracle VM in VirtualBox sounds a bit wrong, why would you like to do server virtualization within desktop virtualization? Well, the performance might not be very good, but being able to play around with Oracle VM, without using several physical servers, sounds like a good idea to me.

In July 2012 Oracle released a White Paper called ["Oracle VM 3: Building a Demo Environment using Oracle VM VirtualBox"](http://www.oracle.com/technetwork/server-storage/vm/ovm3-demo-vbox-1680215.pdf). Along with the White Paper, two VirtualBox images were provided, one for OVMServer and one for OVMManager (urls are in the White Paper). Following the step by step description, you will soon have two Oracle VM servers and one Oracle VM Manager up and running. So far so good, but you will notice that you can’t do the NFS mount testing from myserver1 and myserver2, simply because you don’t know the password to log into myserver1 and myserver2. Oh well, we will do that later, the Oracle VM Manager image contains a document with all the credentials you need, so we will simply go back and do the test later.

Oracle VM Manager is up and running, and you log into the Oracle VM Manager console with admin/Welcome1. Next you will do a discover of the Oravle VM servers, and you are asked about the Oracle VM Agent Password. No problem, just check the document with all the passwords. Hey, wait a second, there is nothing about Oracle VM Agent Password in the document. You look further, and there are not any passwords for root or other users on myserver1 and myserver2 either. Since most of the passwords in the document are Welcome1, you try that, how hard could it be? Well, harder than you might think, Welcome1 doesn’t let you discover the servers, and not letting you logging in to the myserverX images as root either.

Finally, with help from [Lab – How to Deploy and Manage a Private Cloud](http://www.oracle.com/technetwork/systems/hands-on-labs/lab-privatecloud-ovm-2042254.html), I found that the password for both Oracle VM Agent and root is ovsroot. While searching for ovsroot later, it seems that ovsroot is the default root passwords for several Oracle VM templates, so maybe this was just obvious for anyone having used Oracle VM before (but on the other hand, if you are familiar with Oracle VM, you might not be within the primary group for the White Paper).

After using the correct password, discovery of the Oracle VM servers was just like a walk in the park, and logging into the images with root was no longer a problem. Next step is to continue with the Oracle VM documentation to complete the basic configuration and then go on and play with some of the Oracle VM templates available.

Even though I did spend some time finding the correct password, I really appreciate the effort that some people have put into writing the White Paper and providing the VirtualBox images, thank you very much!!!

