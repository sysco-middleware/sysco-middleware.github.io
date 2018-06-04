---
layout: post
title: Use HTML Content with UMS Adapters in OSB 12c
categories: osb messaging
tags: [OSB, UMS]
author: cliops
---
In this post, we will learn how to send html content for the emails using a UMS adapter in OSB 12c. There we go:

### Pre-requirements ###

First, we need to verify that we have the UMS adapter and the email driver already working. These configurations are in the previous post [Using UMS Adapter to send emails with OSB 12c](http://blog.sysco.no/osb/messaging/UMS_Adapter_send_email_OSB_12C/).

### Steps ###

So first we have to edit the UMS adapter and go to the step 5, Set the option "Message is Opaque (Base64Binary)"

 ![](/images/2016-11-27-UMS_Adapter_send_html_content_email_OSB_12C/Image1.JPG)

Now, we need to encode the information to Base64 and use use it in the businees service. To accomplish this task we could use a Java callout and use it to encode the information. I have created a Jar file that encode and decode as Base64 and you can download it [here](/files/libraries/jar.rar).

Extract the jar library and import into the SB Project

![](/images/2016-11-27-UMS_Adapter_send_html_content_email_OSB_12C/Image2.JPG).

Create a Java Callout in the pipeline and use the Jar file called decodejarfile.jar as the image bellow.

![](/images/2016-11-27-UMS_Adapter_send_html_content_email_OSB_12C/Image3.JPG).

Select the method encode from sysco.binary64.Base64Encoder as the following image.

![](/images/2016-11-27-UMS_Adapter_send_html_content_email_OSB_12C/Image4.JPG)

Create a sample html content in the input.

![](/images/2016-11-27-UMS_Adapter_send_html_content_email_OSB_12C/Image5.JPG)

And set a output parameter, let say htmlEncodedContent

![](/images/2016-11-27-UMS_Adapter_send_html_content_email_OSB_12C/Image6.JPG)

Replace the content body with the request of the UMS business service as the image bellow.

![](/images/2016-11-27-UMS_Adapter_send_html_content_email_OSB_12C/Image7.JPG)

Finally, go to routing and include a transport header. Add the following header field name "jca.ums.part.content-type" and the value 'text/html; charset=utf-8' as the image bellow.

![](/images/2016-11-27-UMS_Adapter_send_html_content_email_OSB_12C/Image8.JPG)
This setting will be used to handle html content in the email.

Deploy changes and enjoy sending email with HTML content. Good luck!
