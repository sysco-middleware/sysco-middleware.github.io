---
layout: post
title: How to call web service that uses WS-Addressing from OSB 12c
categories: osb jdeveloper
tags: [Oracle, Oracle Service Bus, JDeveloper, WS-Addresing, WSA, OSB 12c, SoapUI]
author: denzza
---

One of the external systems is using web services with WS-Addressing to retrieve Login information for the user in this case it is a token number. After which this token number will be used when calling other web services from that system.

So to call this web service from Oracle Service Bus 12c we need to create correct SOAP headers to enable the WS-Addressing callout.

Short overview of WS-Addressing
--------------

WS-Addressing provides a transport-neutral mechanism to address Web services and their associated messages. Using WS-Addressing, endpoints are uniquely and unambiguously defined in the SOAP header.
WS-Addressing provides two key components that enable transport-neutral addressing, including:

+	**Endpoint reference (EPR)** - Communicates the information required to address a Web service endpoint.
+	**Message addressing properties** - Communicates end-to-end message characteristics, including addressing for source and destination endpoints and message identity, that allows uniform addressing of messages independent of the underlying transport.

*Example message with WS-Addressing - Request Message:*

![](/images/2016-12-22-How_to_call_web_service_that_uses_WS-Addressing_from_OSB_12c/WS-AddressingSampleMessage.png)

Example of SoapUI call to the web service with WS-Addressing.
--------------

For this example I will use SoapUI 5.2.1. Calling a service with WS-A from SoapUI is fairly simple, just configure all necessary Message addressing properties for the SOAP Headers. In my case which worked for me, you can see in the image below. That options you can find by clicking on the tab “WS-A” beneath the request message window.

![](/images/2016-12-22-How_to_call_web_service_that_uses_WS-Addressing_from_OSB_12c/WS-AddressingSoapUI.jpg)

Here you can also see a simple request message that I will be using trough this post.
Now when all is configured we can call the service.

![](/images/2016-12-22-How_to_call_web_service_that_uses_WS-Addressing_from_OSB_12c/WS-AddressingSoapUI_Response.jpg)

Now the service call was successful and the response gave as a Token, which was essentially the objective.
As you can see this was pretty easy to do with SoapUI, now we need this exact response in Oracle Service Bus 12c.

Web service callout with WS-A from OSB 12c
--------------

To do a WS-A service call on Service Bus 12c we need to replicate the Message addressing properties in the Service Callout from the pipeline of our project.
First step is to create a Business Service from the web service’s WSDL and URI address. Then we can create a Service Callout in the pipeline of our Oracle Service Bas 12c project. In the Service Callout we select the Business service just created and chose the correct operation that will be used in case you have more than one.

Next step is to select the ‘Configure Body’ as the configuration option and create mandatory variables for Body Request/Response and plus create variables for Header Request/Response. Header variables will be used to add the SOAP headers used for WS-Addressing call.

![](/images/2016-12-22-How_to_call_web_service_that_uses_WS-Addressing_from_OSB_12c/WS-Addressing_OSB12c.jpg)

As a last step add Assign activities for the Header and Body request variables.

![](/images/2016-12-22-How_to_call_web_service_that_uses_WS-Addressing_from_OSB_12c/WS-Addressing_OSB12c_2.jpg)

![](/images/2016-12-22-How_to_call_web_service_that_uses_WS-Addressing_from_OSB_12c/WS-Addressing_OSB12c_3.jpg)

LoginWSA_ReqHeader:

```xml
<soap-env:Header xmlns:wsa="http://www.w3.org/2005/08/addressing" xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
	<wsa:Action soap:mustUnderstand="1">http://tempuri.org/AuthenticationService/LogInUser</wsa:Action>
	<wsa:To soap:mustUnderstand="1">http://websrv1u.com/SecurityES.WS/services/Security.ServiceFacade.AuthenticationService.svc</wsa:To>
</soap-env:Header>
```


LoginWSA_RequestBody:

```xml
<soapenv:Body xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://tempuri.org/">
	<tem:LogInUser>
	 	<tem:userName>OSB12c</tem:userName>
		<tem:password>Pass@25462</tem:password>
	</tem:LogInUser>
</soapenv:Body>
```

Optional:
If you need to add some more properties you can use Transport Header activity and add it as http Content-Type property inside the header for example:
Content-Type:

'application/soap+xml;charset=UTF-8;action="http://tempuri.org/AuthenticationService/LogInUser"'

Now your final service callout should look like this:

![](/images/2016-12-22-How_to_call_web_service_that_uses_WS-Addressing_from_OSB_12c/WS-Addressing_OSB12c_4.jpg)

Now you can deploy and test your Service Bus 12c project and service call with WS-Addressing.
For this example the WS-A service response looks like this:

![](/images/2016-12-22-How_to_call_web_service_that_uses_WS-Addressing_from_OSB_12c/WS-Addressing_OSB12c_5.jpg)



[1]: http://www.darkroastedblend.com/2007/01/stars-planets-scale-comparison.html
[2]: http://www.complex.com/pop-culture/2013/04/gallery-babies-using-technology/9
[3]: http://www.thatjeffsmith.com/data-modeling/
[4]: http://docs.oracle.com/cd/E37547_01/tutorials/tut_ide/tut_ide.html
[5]: http://www.quickmeme.com/meme/3rkpgw
