---
layout: post
title: Customizing service calls with multiple levels of security in OSB 12c
categories: osb jdeveloper
tags: [Oracle, Oracle Service Bus, JDeveloper, File Transfer, Security, Client Certificate, Authentication, OSB 12c]
author: denzza
---

This will be a part two of my previous post which you can read [here](http://blog.sysco.no/osb/jdeveloper/Service-call-with-multiple-levels-of-security-in-OSB-12c/ "Service call with multiple levels of security in OSB 12c"). In this post I will talk about customization part of the solution for service calls with different levels of security in OSB 12c. So, we had to develop two different solutions ei. two different service calls to different environments with different levels of security.

Let see what was the requirement in the first place, this is part of the previous post but just for a refresh:
+ In one of the Oracle Service Bus 12c integrations we had system which had different security for different environments for its services. So, for Development environment there was a simple Basic authentication (Username/Password combo) which was theirs Test environment also. Then in their Production environment they had Basic authentication + Client Certificate authentication combined. So, only two environments with two different set of security and there was no option for us to test any kind of solution in theirs one and only Dev/Test/QA environment which from now on I will refer as PREPROD. We had to do it directly in their PROD environment.

So, basically what we have here is that every service call has a Proxy service as initial call. After that proxy calls Business service or in other cases calls a Pipeline. But even those business services were different. We had to develop PREPROD_Proxy and PREPROD_Business service for each service call. And then the same for PROD, we created PROD_Proxy and PROD_Business. 
This means that you will need to replace PREPROD_Proxy for PROD_Proxy in the customization file. 
And this can be done in the **ReferenceCustomizationType** element in your customization file:
```xml
<cus:customization xsi:type="cus:ReferenceCustomizationType">
```

This element is usually generated last in the customization file and here you can find your Proxy and do the change easily as **externalReferenceMap** element. For example:

```xml
<cus:externalReferenceMap>
    <xt:oldRef>
      <xt:type>ProxyService</xt:type>
      <xt:path>Testing_Security_Customization/Proxy/PutFiles_to_External_PS_PREPROD</xt:path>
    </xt:oldRef>
    <xt:newRef>
      <xt:type>ProxyService</xt:type>
      <xt:path>Testing_Security_Customization /Proxy/PutFiles_to_External_PS_PROD</xt:path>
    </xt:newRef>
</cus:externalReferenceMap>
```
And the proxy will be replaced in the entire OSB project. But there is also an option if you want this to be replaced in a specific Pipeline. 
As a last entry in the customization file just add the same element with the xsi:type="cus:ReferenceCustomizationType" to override all the changes above.
And that would look something like this:
```xml
<cus:customization xsi:type="cus:ReferenceCustomizationType">
    <cus:description/>
    <cus:refsToBeConsidered xsi:type="xt:ResourceRefType">
      <xt:type>Pipeline</xt:type>
      <xt:path>Testing_Security_Customization /Proxy/Testing_Security_Customization</xt:path>
    </cus:refsToBeConsidered>
      <cus:externalReferenceMap>
       <xt:oldRef>
         <xt:type>ProxyService</xt:type>
         <xt:path>Testing_Security_Customization/Proxy/PutFiles_to_External_PS_PREPROD</xt:path>
       </xt:oldRef>
       <xt:newRef>
         <xt:type>ProxyService</xt:type>
         <xt:path>Testing_Security_Customization /Proxy/PutFiles_to_External_PS_PROD</xt:path>
       </xt:newRef>
    </cus:externalReferenceMap>
  </cus:customization>
```

And this is how we change the Proxy Endpoint in the Service Callout or Publish or any kind of activity inside the pipeline of the OSB 12c project.

Next thing we have to customize is as you remember authentication for two environments and different credentials. These credentials are used in the Transport Headers activity as you can see in [previous post](http://blog.sysco.no/osb/jdeveloper/Service-call-with-multiple-levels-of-security-in-OSB-12c/ "Service call with multiple levels of security in OSB 12c") and it looks like this:


![](/images/2018-03-11-Customizing-service-calls-with-multiple-levels-of-security-in-OSB-12c/03_TransportHeadersAutorization.jpg)

Here is why I used the variable $AutorizationVar instead of hardcoding the credentials, Customization. To be able to customize a variable value we can use this trick with the xQuery.
Create a new xQuery files where you will add content with your authentication credentials:

![](/images/2018-03-11-Customizing-service-calls-with-multiple-levels-of-security-in-OSB-12c/AuthenticationxQuery.jpg)

Now to assign this value to the variable from xQuery use Assign activity.

![](/images/2018-03-11-Customizing-service-calls-with-multiple-levels-of-security-in-OSB-12c/AuthenticationxQueryAssign.jpg)

And as a last step we will update the Customization file the same way as above for replacing the Proxy.
```xml
<cus:externalReferenceMap>
      <xt:oldRef>
        <xt:type>Xquery</xt:type>
        <xt:path>Testing_Security_Customization/Mapping/AutorizationTest</xt:path>
      </xt:oldRef>
      <xt:newRef>
        <xt:type>Xquery</xt:type>
        <xt:path>Testing_Security_Customization/Mapping/AutorizationProduction</xt:path>
      </xt:newRef>
    </cus:externalReferenceMap>
```
And now you are set and can deploy your integration and test your customization. 


Customization and replacement of Business with Proxy service with Dynamic Routing
------------------

This is an additional customization where I needed to use Routing inside my Pipeline and I wanted to use Business service call for DEV and Proxy service call for PROD. Here I had couple of things to customize. First, replace Business service with the Proxy is not possible in the same standard way in the Customization file as shown above. 
Here I had to customize:
+	Authentication – Done and explained above
+	URI customization – Same method as for Authentication
+	Customization and replacement from Business to Proxy service used in Routing – Focus is on this implementation.


To replace Business service with Proxy service in customization file seems not possible. As I needed to use Routing for this. Then I realized that I can actually use Dynamic Routing and in that way by manipulating the Service element I can change and replace any type of links and endpoints.
This is done by using this example:
```xml
<ctx:route>
<ctx:service isProxy=’false’>absolute path of business service</ctx:service>
<ctx:operation>operation name</ctx:operation>
</ctx:route>
```

But I didn’t need the operation so I removed that element and this is how I used this example to customize by my needs.
```xml
<ctx:route>
<ctx:service isProxy='{$isProxy}'>{$ExternalEnvironmentVar}</ctx:service>
</ctx:route>
```

![](/images/2018-03-11-Customizing-service-calls-with-multiple-levels-of-security-in-OSB-12c/DynamicRoutingCustomization.jpg)

I had to customize both of the variables because I’m replacing the Business to Proxy service. This is done in the same way as variable for Authentication explained above. 

![](/images/2018-03-11-Customizing-service-calls-with-multiple-levels-of-security-in-OSB-12c/isProxyxQuery.jpg)

![](/images/2018-03-11-Customizing-service-calls-with-multiple-levels-of-security-in-OSB-12c/BusinessToProxyxQuery.jpg)

Maybe there is more elegant way in resolving this type of replace and customization from Business to Proxy services or vice versa but this is what first worked in my case.  


[1]: http://www.darkroastedblend.com/2007/01/stars-planets-scale-comparison.html
[2]: http://www.complex.com/pop-culture/2013/04/gallery-babies-using-technology/9
[3]: http://www.thatjeffsmith.com/data-modeling/
[4]: http://docs.oracle.com/cd/E37547_01/tutorials/tut_ide/tut_ide.html
[5]: http://www.quickmeme.com/meme/3rkpgw
