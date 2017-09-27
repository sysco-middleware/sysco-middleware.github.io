---
layout: post
title: Archiving files on request based on files name with FTP Adapter in OSB 12c
categories: osb jdeveloper
tags: [Oracle, Oracle Service Bus, JDeveloper, File Transfer, PDF Transfer, Archive, Document Transfer by name, OSB 12c]
author: denzza
---

This as an additional requirement from one of my previous posts which you can read [here](http://blog.sysco.no/osb/jdeveloper/Get-file-on-request-based-on-a-file-name-and-send-it-via-HTTP-POST-with-OSB-12c/ "Get file on request based on the file name.").

But for those who are interested only in this part of the solution this is a short recap of the previous post requirement:
*Requirement is fairly simple, there is a need to transfer a “non XML” file (in this specific case .pdf document) based on the file name which is sent in the Request message and transfer the file to another external system using HTTP POST.*

Now there is a need for that file to be transferred and archived in different folder in the same folder structure on the same server. This means that we basically need to cut and paste the file/document from current folder to another which is one level down in folder structure.
We can have two scenarios here with the “Archive” action:
 
+	**Copy-Paste scenario:** Archive file without deleting it from current folder and pasting it to the other folder. 
+	**Cut-Paste scenario:** Which I will be using in this post is, Archive file with deleting it in the current folder and pasting it to another.  

As I used the FTP adapter to fetch the file from the server based on the file name I thought: “Ok this will be easy and straightforward.” 

Why did I think that? 
Well the FTP Adapter has an option in “File Directories” wizard step where you can set up if you want your processed file to be archived (image below). You check Archive processed files and provide a new physical path for the archive folder. Straightforward right?

![](/images/2017-09-27-Archiving-files-on-request-based-on-files-name-with-FTP-Adapter-in-OSB-12c/FileDirectoryWizzard.jpg)

Well… Let’s just say things escalated quickly… During the deployment of the integration I got this error message:

**[ERROR] The session cannot be activated due to existence of conflicts.
resource: BusinessService ArchiveTestIntegration/Business/ArchiveFile
    error: Invalid JCA transport endpoint configuration, exception: BINDING.JCA-11000
Invalid Activation parameter.
Activation parameter Physical/Logical ArchiveDirectory has invalid value {/Internal/in/archive}.
Please correct the activation parameter and redeploy the process.**

So I started to test a little bit: 

+	Physical path was correct but I still played a little bit with the path adding the full file structure of the server starting from the root folder. No changes, same error.
+	I couldn’t use any other option except – Synchronous Get File (because of the get file by the file name requirement)
+	Tried with different folder with different permission, got the same error.
+	Then I tried with physical path of the folder in the weblogic where my integration is deployed and… it worked. Hmm…

This was actually my first time using archive option in the FTP adapter and my understanding was that this will work on the FTP server itself from which I’m retrieving the files.
So with FTP adapter you can GET, you can PUT the files but you can’t “Archive” the files on the same FTP server? 
At least not for the Synchronous Get File option: I haven’t tried with simple Get File option in FTP Adapter.

![](/images/2017-09-27-Archiving-files-on-request-based-on-files-name-with-FTP-Adapter-in-OSB-12c/FTPAdapterOperationPage.jpg)

Ok then, now for the solution part. Only thing I could do here was to split the FTP call in two.
Decouple the GET and Archive part of the operation. So, in first call I retrieve the file do some other operation and transfers if everything is successful then I call the second business service which is a FTP Transfer. 
In the first GET operation I set the file to be deleted upon successful retrieval.  

![](/images/2017-09-27-Archiving-files-on-request-based-on-files-name-with-FTP-Adapter-in-OSB-12c/AdapterProperties.jpg)

And in second Business Service I do a simple “Put file” in the Archive folder with FTP Transfer as an external service. 


[1]: http://www.darkroastedblend.com/2007/01/stars-planets-scale-comparison.html
[2]: http://www.complex.com/pop-culture/2013/04/gallery-babies-using-technology/9
[3]: http://www.thatjeffsmith.com/data-modeling/
[4]: http://docs.oracle.com/cd/E37547_01/tutorials/tut_ide/tut_ide.html
[5]: http://www.quickmeme.com/meme/3rkpgw
