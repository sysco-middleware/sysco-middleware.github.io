---
layout: post
title: Service call with multiple levels of security in OSB 12c
categories: osb jdeveloper
tags: [Oracle, Oracle Service Bus, JDeveloper, File Transfer, Security, Client Certificate, Authentication, OSB 12c]
author: denzza
---

![Client Certificate security implementation](/images/2017-09-27-Service-call-with-multiple-levels-of-security-in-OSB-12c/HTTP_GET_Security.jpg)

In one of the Oracle Service Bus 12c integrations we had system which had different security for different environments for its services. So, for Development environment there was a simple Basic authentication (Username/Password combo) which was Test environment also. Then in their Production environment they had Basic authentication + Client Certificate authentication combined. So, only two environments with two different set of security and there was no option for us to test any kind of solution in theirs one and only Dev/Test/QA environment which from now on I will refer as PREPROD. We had to do it directly in theirs PROD environment.

So, customization between environments became little bit complicated where we had to implement different solutions for those two environments for every service call to that system. But about customization of these calls I will write in separate post. 

Implementation of the PREPROD call was straightforward nothing fancy. But for calls to PROD we had different implementation for different service calls. For example we had a simple WSDL based services calls then we had a file transfer one HTTP POST and other HTTP GET based on the provided URI. And here with the POST and GET is where situation became little tricky.

So first let’s start with HTTP POST security implementation. Because of two levels of security (Basic user/password and additional Client Certificate security) we had to use Proxy and Business service together. Prior any work in OSB you have to add your Client Certificate into your Identity Key Store, which I will not go into details but while setting it up just remember to pick up the correct alias name for the certificate. When you have that part sorted you create a Service Key Provider in your OSB 12c project where you will use that alias name for the Service Key Provider which looks something like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ser:credentials xmlns:ser="http://www.bea.com/wli/sb/services">
  <ser:key-pair purpose="SSL">
    <ser:alias>mysystem-alias-ssl-client-production-keystore</ser:alias>
    <ser:password>randompa55</ser:password>
  </ser:key-pair>
</ser:credentials>
```
From the pipeline we would call the Proxy service first which will have the Security Configuration set for the Service Key Provider which then calls the Business Service which will have the “Client Certificate” as Authentication option. Make sure that in the Proxy service in Transport Configuration  tab you have checked the “Get All Headers” option. This is how Proxy should look:

![](/images/2017-09-27-Service-call-with-multiple-levels-of-security-in-OSB-12c/01_ProxyTransportDetails.jpg)

![](/images/2017-09-27-Service-call-with-multiple-levels-of-security-in-OSB-12c/02_ProxySecurityjpg.jpg)

And this is how Business service should look:

![](/images/2017-09-27-Service-call-with-multiple-levels-of-security-in-OSB-12c/03_TransportConfiguration.jpg)

As you noticed we are still missing Basic authentication… This can be done by transferring the Username/Password trough the Transport Headers activity. So, for the actual call from the pipeline we will use Service Callout and here add a Transport Headers activity with these headers (in my case I’m working with the pdf files so this will have an additional header for Content-Type):

![](/images/2017-09-27-Service-call-with-multiple-levels-of-security-in-OSB-12c/04_TransportHeadersAutorization.jpg)

Where **@AutorizationVar** will have a Base64 encoded value for the Username/Password and it will look something like this: 

“**Basic b9yZVzZ3J1cHBlbjGa1Y1jdDlZ5NWtOemF6NUF4Q==**” 

You will have to encode your credentials first so you can use it in this format.


### To recap ###

![](/images/2017-09-27-Service-call-with-multiple-levels-of-security-in-OSB-12c/HTTP_POST_security.jpg)

+	From the pipeline we use Service Callout where we add a Transport Headers for Basic authentication.
+	Service Callout then calls a Proxy Service which adds a Service Key Provider and transfers Basic credentials from the header to the Business service which it invokes.
+	Business service has a Client Certificate security option as an additional level security plus all headers that came from the Proxy service. 

So as for the HTTP POST call you are set to go. 

Now it’s HTTP GET turn, and you would assume that this is done the same as the POST part but it’s a little bit different. For the GET part we have an URI from which we are retrieving the files and for not sure which reason this could not be done as Proxy to Business call. 
Example of the URI:


**https://preprod.external.com/doc/clientname/docid/0908201750ox0euz01ps7gs77tpi0tj6o**

For URI transfer we had to use Routing Options activity where we set up the URI. But URI never gets transferred from Proxy to Business service. So, to make this work in combination with two levels of security we had to add another pipeline in between Proxy and Business service.
In this additional pipeline we would add URI Routing option. 
Proxy and Business services in this implementation can’t be invoked with Service Callout or Publish, here we need to use Routing. And here is how the implementation would look:

![](/images/2017-09-27-Service-call-with-multiple-levels-of-security-in-OSB-12c/HTTP_GET_Security.jpg) 


[1]: http://www.darkroastedblend.com/2007/01/stars-planets-scale-comparison.html
[2]: http://www.complex.com/pop-culture/2013/04/gallery-babies-using-technology/9
[3]: http://www.thatjeffsmith.com/data-modeling/
[4]: http://docs.oracle.com/cd/E37547_01/tutorials/tut_ide/tut_ide.html
[5]: http://www.quickmeme.com/meme/3rkpgw
