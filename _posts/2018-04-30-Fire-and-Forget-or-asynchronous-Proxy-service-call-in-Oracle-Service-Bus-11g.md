---
layout: post
title: "Fire and Forget" or asynchronous Proxy service call in Oracle Service Bus 11g
categories: osb jdeveloper
tags: [Oracle, Oracle Service Bus, JDeveloper, OSB 11g, Fire and Forget, Asynchronous Call]
author: denzza
---

![Fire and Forget call in OSB 11g](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/AsynchronousCall_OSB11g.jpg)
----------------

#####Requirement:#####

As an requirement client wanted to trigger the Integration (Oracle Service Bus 11g) in "Fire and Forget" manner or asynchronous manner. Just the option to trigger the OSB to start executing without any waiting for process to finish. 
In our case we had an integration which did some processing of data from the database couple of times and saving it to the Topic which took some time (in this case just a couple of minutes). But that was not something that the client side wanted to wait to finish and it didn’t need to wait. Specification was that they want to trigger our proxy service in the scheduled times. 


#####Solution:#####
We already had implemented logic in the Proxy Service which takes some time to finish the process. Reason for that is that we had to call DB multiple times and it had a lot of data. So, if we call our Proxy Service directly we would need to wait until whole process is complete to get a response back. We know that asynchronous call can be achieved with additional Queue as a recommended solution in OSB 11g, but we didn’t want to have another one, because our integration already has one JMS queue and one topic. 
First thing I have tried is to call our Proxy with another Proxy from the same project. But that of course didn’t work. 

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/AsynchronousCall_OSB11g_Fail.jpg)

But, then I created additional Business Service in between these two Proxy services which worked for us and did the desired effect. 

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/AsynchronousCall_OSB11g.jpg)

Now I will show how did I simply set my Proxy and Business services in OSB 11g. First let’s start with the Business service which was really simple. 
For Service Type I used **Messaging Service** and for Request Message Type: **Text** and Response Message Type: **None**. As last thing is to set Endpoint URI to point to your Proxy service you want to invoke (in our case that is long running Proxy). The rest of the setup is default.

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/BusinessServiceTrigger_1.jpg)

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/BusinessServiceTrigger_2.jpg)

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/BusinessServiceTrigger_3.jpg)

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/BusinessServiceTrigger_4.jpg)


For the Proxy service I used a custom made WSDL so I can trigger this from SoapUI as well. But it should work with the Messaging Service option as well because we don’t use any data in the request. 

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/ProxyServiceTrigger_1.jpg)

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/ProxyServiceTrigger_2.jpg)

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/ProxyServiceTrigger_3.jpg)

In the message flow all you need is a Publish activity to call our business service we created first. 

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/ProxyMessageFlow.jpg)

But, to mimic OK response back to the client and that the service was actually called we have to add two Insert activities in the Response pipeline of our Proxy.
First insert should add the http response code in the response element of the transport variable.

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/Instert_1.jpg)

Second insert should add the text to the response message.

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/Instert_2.jpg)

Your integration is now ready for deployment and a test.


#####Test:#####

To test the trigger from the SoapUI create the SoapUI project and run the test:

![](/images/2018-04-30-Fire-and-Forget-or-asynchronous-Proxy-service-call-in-Oracle-Service-Bus-11g/SoapUI_Testing.jpg)

You will not get the response in the XML instead you have to click on the Raw tab to see the response header from the service and here you see the response was OK.

#####Triggering with cURL command:#####

Our requirement was to call (trigger) the OSB in scheduled times during the day. We found that cURL command will do the job for us and with cURL it can be triggered from the external system which was responsible for triggering schedule. I will not go into details of curl command but in this case should be self explanatory:

**curl -X POST http://localhost/int/TestPro/ProxyServiceTrigger --data-binary "@../request.xml" --header "SOAPAction: process"  -H "Content-Type: text/xml;charset=UTF-8"**

request.xml file from the command has the content:

```xml
<soapenv:Envelope xmlns:phar="http://xmlns.oracle.com/int_TestPro/TriggerInstructionGenerator_v1/InstructionGeneratorProcess" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
	<soapenv:Header>
		<wsse:Security soapenv:mustUnderstand="0" xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd">
			<wsse:UsernameToken wsu:Id="UsernameToken-34141130" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">
				<wsse:Username>TestUser</wsse:Username>
				<wsse:Password Type="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-username-token-profile-1.0#PasswordText">435e678af7b2245ef0f15412f</wsse:Password>
			</wsse:UsernameToken>
		</wsse:Security>
	</soapenv:Header>
	<soapenv:Body>
		<phar:process>
		</phar:process>
	</soapenv:Body>
</soapenv:Envelope>
```

Save the file where the cURL command will be executed from and has access to.
 


[1]: http://www.darkroastedblend.com/2007/01/stars-planets-scale-comparison.html
[2]: http://www.complex.com/pop-culture/2013/04/gallery-babies-using-technology/9
[3]: http://www.thatjeffsmith.com/data-modeling/
[4]: http://docs.oracle.com/cd/E37547_01/tutorials/tut_ide/tut_ide.html
[5]: http://www.quickmeme.com/meme/3rkpgw
