---
layout: post
title: Handle dynamic params in metadata for REST connectors in OSB 12C
categories: OSB tips
tags: [REST, OSB]
author:
- cliops
- dalibor
---
In this post, we will learn how to get the value of some path parameters from the URI of a REST Service using OSB 12C. There we go:

### Use case ###

First, lets think that we have to consume a sample REST service to create one employee in a specific organization, so we will have a dynamic path parameter after organizations that we are going to send in the URI.

URI:
- https://RestServices.com/api/organizations/1/employees

Method:
- POST

ContentType:
- application/json

ContentBody

```javascript
 {         
   "employeeNumber":123,
   "fullName":"Christopher Laurente",
   "Age":27,
   "Country":"Norway"
   ....
  }

```

Then we start, open JDeveloper (In my case is 12.2.1), create a Service bus project and add a Rest component to External services in the composite as the image below.

![](/images/2017-09-18-Rest-connector-Path-params-OSB-12C/Image1.jpg)

Set the name, for instance CompaniesAPI and check the box "Reference will be invoked by components using WSDL interfaces".

![](/images/2017-09-18-Rest-connector-Path-params-OSB-12C/Image2.jpg)

Set the Base URI as https://RestServices.com/api and add a Resource Path as organizations/{organization_id}/employees. See the image below.

![](/images/2017-09-18-Rest-connector-Path-params-OSB-12C/Image3.jpg)

Then create a Operation Binding, call it CreateEmployee and select the HTTP Verb as POST.

![](/images/2017-09-18-Rest-connector-Path-params-OSB-12C/Image4.jpg)

Now, we need to reference the path param called organization_id, for this we are going to use the variable $property according to oracle documentation
https://docs.oracle.com/middleware/1213/osb/develop/GUID-C346DF7D-041D-4E10-BE1C-451F50719106.htm#OSBDV88240

![](/images/2017-09-18-Rest-connector-Path-params-OSB-12C/Image5.jpg)

The variable called organizationId will be set in the metadata of the outbound request as the following structure:

```xml
<ctx:transport>
 <ctx:request>
   <headers …>
   <ctx:user-metadata name="organizationId" value="1" />
 </ctx:request>
</ctx:transport>
```

One of the challenges that I got was that I could not insert the metadata directly into the outbound variable, because I got CData error everytime that I tried. So what I did was assigning the whole value of the outbound in an auxiliar variable, insert the metadata into that aux variable and replace the outbound with the value of the auxiliar outbound.

Assign to auxiliar variable
![](/images/2017-09-18-Rest-connector-Path-params-OSB-12C/Image6.jpg)

Insert metadata
![](/images/2017-09-18-Rest-connector-Path-params-OSB-12C/Image7.jpg)

Replace Outbound
![](/images/2017-09-18-Rest-connector-Path-params-OSB-12C/Image8.jpg)

### Conclusion ###

I'm gonna say that I had to review oracle documentation to understand how to use this REST component correctly and not always will be necessary to use this solution to send dynamic params in the URI (even for query params), because if is possible to use the content body to send these params then it will be straightforward to manage them rather than the solution that I described above.
