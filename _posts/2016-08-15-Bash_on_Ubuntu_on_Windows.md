---
layout: post
title: Bash on Ubuntu on Windows
categories: exalogic
tags: [windows, windows 10, linux, ubuntu, ssh]
author: pah
---

Simplifying ssh login to Linux by using Bash on Ubuntu on Windows 10

Earlier you had to install/use third party software in Windows to run ssh. The new Bash on Ubuntu on Windows gives Windows new built in possibilities.  Under is an explanation on howto setup and login into a machine in an organization which is behind a jumphost. Earlier you typically used putty, cygwin, etc.. for this.
 

The machines in the example are called clienthost, jumphost and serverhost.


*Clienthost --> your Windows machine*
*Jumphost   --> the machine in the middle*

*Serverhost --> the machine which you are going to log into*



Earlier you typically logged into the machines like this:

 
*clienthost --> jumphost --> serverhost*

Eks (putty)

![](/images/2016-08-15-Bash_on_Ubuntu_on_Windows/putty_ssh_01.png)


```bash
login as: myuser
myuser@jumphost's password:
Last login: Thu Aug 15 10:46:27 2016 from clienthost.example.com
[myuser@jumphost ~]$ 

```

Or in one line (cygwin)

```bash
myuser@clienthost /home/myuser # ssh -t myuser@jumphost ssh myuser@serverhost
myuser@jumphost's password:
myuser@serverhost's password:
[myuser@serverhost ~]# 

```

You can also do the last one in Bash on Ubuntu for Windows.
 
  - The -t option is to force pseudo-tty allocation
 
So how can we simplify this. This is nothing new in windows bash. Just to exploit the new possibilities. Start the bash shell on Ubuntu on Windows.

![](/images/2016-08-15-Bash_on_Ubuntu_on_Windows/bash_ssh_01.png)

Edit the file config and add info about jumphost and serverhost like this:

```bash
myuser@clienthost /home/myuser # cd ~/.ssh
myuser@clienthost /home/myuser/.ssh # vi config

*# Jumphost (this is the proxyhost)
Host jumphost
User myuser
Hostname jumphost.example.com
 
Host serverhost
Hostname serverhost.example.com
Port 22
User myuser
ProxyCommand ssh jumphost -W %h:%p *

```

Now you are using jumphost as a proxy to log directly into serverhost. Everything behind Host is an alias and you can call it whatever you want. Behind Hostname there must be a dns-name or IP-address. User is the user you want to log in with. Now you log into serverhost like this:

```bash
myuser@clienthost /home/myuser # ssh el01cn01
myuser@jumphost's password:
myuser@serverhost's password:
[myuser@serverhost ~]#

```

To simplify it even more you create ssh keys on your client and distribute the public-keys to the jumphost and the serverhost. 

Now you do all this from Windows without any third party software.



