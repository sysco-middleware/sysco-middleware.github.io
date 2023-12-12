---
layout: post
title: Setup OTD HA
categories: exalogic
tags: [exalogic, otd, oracle, ha, Oracle Traffic Director]
author: pah
keep: yes
---

OTD active/standby failover

You have installed OTD (Oracle Traffic Director), configured nodes and
created an instance. Now you want to set up an OTD active standby
failover. First you will need a virtual IP address. Log in to EMOC
(Enterprise Manager OPS Center) and allocate a virtual IP for use in the
OTD failovergroup.

Go to the accounts Networks setting and press the icon Allocate vIP from
vNet in the section Public Networks.

![](/images/2015-05-08-setup-otd-ha/otd_ha_VIP1.png)

Choose your number of vIPs you want.

![](/images/2015-05-08-setup-otd-ha/otd_ha_VIP2.png)

Make a note of the IP-address given. This is the failoveraddress you
will use in OTD.

![](/images/2015-05-08-setup-otd-ha/otd_ha_VIP3.png)

Add the virtual IP allocated to /etc/hosts on all VM's. If you don't
have a DNS server inside the exalogic, this is to make sure the traffic
remains inside the exalogic.

```bash
[root@admvm \~]\# vi /etc/hosts
192.168.99.99 soa-exaqa.xxx.yy \# OTD virtual http listener address

```

Update all the hosts in this environment with the new line in
/etc/hosts.

```bash
[root@admvm \~]\# scp -rp /etc/hosts wlsvm2:/etc/hosts
root@wlsvm2's password:
hosts 100% 586 0.6KB/s 00:00
[root@admvm \~]\# ssh wlsvm1 cat /etc/hosts |grep virtual
root@wlsvm1's password:
192.168.99.99 soa-exaqa.xxx.yy \# OTD virtual http listener
address

```


#### Setup Failovergroup

Now we are at the point setting up the failover.

Click the button New Failover Group to create the failovergroup.

![](/images/2015-05-08-setup-otd-ha/otd_ha1.png)

Type the virtual IP address or hostname (in /etc/hosts) and a unique
Router ID.

![](/images/2015-05-08-setup-otd-ha/otd_ha2.png)

Router ID must be unique. If f.eks router id 254 also is used in another
environment inside exalogic, this will NOT work. If so change the Router
ID to f.eks 253.

In a ssh session, find which bond is holding the public ip and choose
that one in the Network Interface (NIC) section:

```bash
[root@otdvm1 \~]\# ifconfig -a bond3
bond3 Link encap:Ethernet HWaddr 00:24:4E:FA:A3:74
inet addr:192.168.99.97 Bcast:192.168.99.255 Mask:255.255.255.0
UP BROADCAST RUNNING MASTER MULTICAST MTU:1500 Metric:1
RX packets:227253 errors:0 dropped:33798 overruns:0 frame:0
TX packets:903599 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:0
RX bytes:17704581 (16.8 MiB) TX bytes:1234088793 (1.1 GiB)
[root@otdvm2 \~]\# ifconfig -a bond3
bond3 Link encap:Ethernet HWaddr 00:34:4A:FB:3A:64
inet addr:192.168.99.98 Bcast:192.168.99.255 Mask:255.255.255.0
UP BROADCAST RUNNING MASTER MULTICAST MTU:1500 Metric:1
RX packets:247610 errors:0 dropped:33803 overruns:0 frame:0
TX packets:915563 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:0
RX bytes:19190583 (18.3 MiB) TX bytes:1235665926 (1.1 GiB)
```

Pick the Primary Node and Backup Node from the dropdown list, and the
NIC that corresponds with the public ip (here, bond3).

![](/images/2015-05-08-setup-otd-ha/otd_ha3.png)

Press next and Create Failover Group.

![](/images/2015-05-08-setup-otd-ha/otd_ha4.png)

![](/images/2015-05-08-setup-otd-ha/otd_ha5.png)

![](/images/2015-05-08-setup-otd-ha/otd_ha5b.png)

You will now see the Primary and Backup node earlier provided to the
wizzard.

Now change the listener settings so Server Name contains the Vip or
hostname for this Vip.

![](/images/2015-05-08-setup-otd-ha/otd_ha6.png)

Save and Deploy changes

Failover is set up on the 2 otd instanses.

![](/images/2015-05-08-setup-otd-ha/otd_ha7.png)
