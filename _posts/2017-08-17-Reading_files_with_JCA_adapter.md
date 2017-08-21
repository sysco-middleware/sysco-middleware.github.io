---
layout: post
title: Reading files in the right order with JCA FTP adapter
categories: oracle soa osb
tags: [Oracle, OPatch, OSB, SOA, Fusion Middleware]
authors:
- dalibor
- cliops
---

Weblogic server FTP JCA adapter can be used to connect to both FTP and SFTP servers from Java, SOA and OSB applications. Therefore it is important to understand how it is working and how it can be properly tuned.

In this short tutorial we are going to explain how to retrieve files from FTP/SFTP server in the right order based on the file timestamp, by using Weblogic server FTP JCA adapter. This is a common problem when we have to assure ourselves that file with most recent content comes last. Especially is that problem important when files are having exactly the same content that has to be written to the e.g. database, but with no sequence marker or timestamp information within the file.

# Configuring adapter

Beauty of using JCA adapters is that they are easily configurable without requiring any special programing from the client side. Many tools (e.g. JDeveloper tools for SOA/OSB development) have nice wizards to configure them. However not all of the adapter options are available in wizards and some of the options are available only on the server side.

Let's have a look what we need to do on the server side and than on the client side in order to configure adapter properly so that it can download files in the right order

## Server side adapter configuration for timestamp based file retrieval

One important thing to mention is that successful timestamp based retrieval will not be done in clustered environment without doing this step. So first thing we have to do is to log in into Weblogic console as administrators and than from **Domain Structure->Deployments** menu we have to choose **FtpAdapter** in the list of available deployments. On the **Settings for FtpAdapter** page we have to click on **Configuration** and then **Outbound Connection Pools** tab. From that tab we will click on the configuration that we have created to connect to our server.

Now on the very first page of the adapter configuration options, as seen on the picture bellow, we have to replace **ControlDir** option value **${user.dir}** with the value that points to the directory found on the shared mounted drive that is accessible by all cluster members

![](/images/2017-08-17-Reading_files_with_JCA_adapter/ControlDir.PNG)

Fig. 1\. FTP JCA adapter Control directory configuration option

To this directory copy of each file downloaded from server is made. Therefore it is important that server administrators create purging script that will periodically remove this files. This files are consulted by each cluster node in order to see which file has been already downloaded. This is important for example when the files form FTP server are not removed. But this files can be used by adapter to check and compare timestamps with files that are still on the server and not downloaded yet.

## Client side adapter configuration for timestamp based file retrieval

When we have configured server side Control directory we can proceed with client side JCA adapter configuration. This is done by modifiying **\*.jca** files which are main configuration files for JCA adapters. To retrieve files based on timestamp we have to add this two important properties:

```xml
<property name="ListSorter" value="oracle.tip.adapter.file.inbound.listing.TimestampSorterAscending"/>
<property name="SingleThreadModel" value="true"/>
```

First property assures that retrieving of files will be done in right timestamp order (for descending order TimestampSorterDescending class has to be used). Second option suppresses parallelism thus avoiding that potentially one file can be read faster and thus come to the client earlier.

Options can be inserted into the file as seen on the picture bellow.

![](/images/2017-08-17-Reading_files_with_JCA_adapter/OrderProperties.PNG)

Fig. 2\. Setting right FTP JCA adapter properties for timestamp based file retreival

# Testing the concept

To test the configured functionality we have created several files with alphabetic names sorted in descendant order while timestamp value has been sorted ascending. All files had the same size. After logging the downloaded files we have observed that server is providing us files in right timestamp based order.

# Conclusion

By using **ListSortorder** property we can configure many other sorting capabilities including custom ones, but possibility to retrieve files from FTP server based on the file timestamp value is extremely useful especially when we can not deduce from the content of the file the right data order.
