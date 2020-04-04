---
layout: post
title: Activate Windows license in Oracle Cloud Infrastructure (OCI)
categories:  cloud
tags: [oci, oracle cloud, windows, license]
author: catoaune 
---

# Scenario

In a private subnet, you have one or more Windows servers where you need to activate the Windows licenses. Instead of using Microsofts own licensing servers, you have to use Oracle KMS service. My Oracle Support Knowledge Base document ID 2228641.1 "License Activation For Windows Instances" describes the process, but unfortunately it didn't work for us.

What we ended up doing was running the following commands in PowerShell on each server (replace 192.168.1.53 with your servers ip):
```
route add  169.254.169.253 mask 255.255.255.255 192.168.1.53
slmgr /skms 169.254.169.253:1688
slmgr /ato
Get-CimInstance -ClassName SoftwareLicensingProduct | where {$_.PartialProductKey} | select Description, LicenseStatus


Description                                           LicenseStatus
-----------                                           -------------
Windows(R) Operating System, VOLUME_KMSCLIENT channel             1
```
LicenseStatus = 1 is ok (it was 5 before we where able to activate)


