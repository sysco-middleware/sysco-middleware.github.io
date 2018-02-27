---
layout: post
title: Manage Facebook Workplace Account Management API with OSB 12.2.1
categories: OSB tips
tags: [OSB, Workplace, API, Web services, Javascript]
author: cliops
---
In this post, we will learn how to manage the Workplace Account Management API with OSB 12.2.1, there we go:

### INTRODUCTION ###

Workplace Account Management API brings you rest services to manage Workplace users in one organization. To integrate this API with OSB 12.2.1 is necessary to know what is the complete structure of the request and response of the REST services. Since the structure of the content body is a bit tricky to manage then we are going to do some steps with the magic of the new javascript feature that is available in 12.2.1.


### SOLUTION ###


The documentation of the API is [here](https://developers.facebook.com/docs/workplace/account-management/api)  

As we can see in the documentation Workplace API is based on the SCIM-compliant ([System for Cross-domain Identity Management](http://www.simplecloud.info/) )

Basically, the content body of the request of get user will look like this below.

```json

{
  "schemas": [
    "urn:scim:schemas:core:1.0",
    "urn:scim:schemas:extension:enterprise:1.0"
  ],
  "id": 11111111111,
  "userName": "juliusc@example.com",
  "name": {
    "formatted": "Julius Caesar"
  },
  "active": true,
  "emails": [
    {
      "primary": true,
      "type": "work",
      "value": "juliusc@example.com"
    }
  ],
  "addresses": [
    {
      "type": "work",
      "formatted": "Foro Italico",
      "primary": true
    }
  ],
  "urn:scim:schemas:extension:enterprise:1.0": {
    "department": "Headquarters"
  }
}

```

If we want to create a NXSD transformation from this sample json then we will get an error since the fields of the json have colons ":"  that are not accepted as names of xml elements.

![](/images/2018-02-27-Workplace-Account-Management-API-with-OSB-12C/Image1.jpg)

Then one of the workarounds is handle the request as string using xquery, for example:

![](/images/2018-02-27-Workplace-Account-Management-API-with-OSB-12C/Image2.jpg)

When we have the request complete as string the next step is call the rest service and set is as JSON using transport header.

![](/images/2018-02-27-Workplace-Account-Management-API-with-OSB-12C/Image3.jpg)

If we want to handle the request or response as Json and not as String in the pipeline then there is a good feature added in OSB 12.2.1 called Javascript located in the Message Processing section.

![](/images/2018-02-27-Workplace-Account-Management-API-with-OSB-12C/Image4.jpg)

Basically, this new component allow us to insert javascript code where we can create or modify existing variables in the pipeline. For example if we want to modify the value of a field in the Json request, then we initiate with the variable called "process" and then just after the name of the existing variable. In this case the variable of the request is a string so it is parsed to Object JSON, changed the active field to false (deactivate user) and finally changing the data type to string as it was before.

![](/images/2018-02-27-Workplace-Account-Management-API-with-OSB-12C/Image5.jpg)



### Conclusion ###

There are several solutions to achieve this integration, like for example using java callout or only handling the request and response as TEXT in the business service. So this workaround that I provide fits more with the new feature of 12.2.1 (javascript) and is not complex at all to complete the integration.


If you have any questions let me know, thanks.
