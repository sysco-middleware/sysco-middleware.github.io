---
layout: post
title: Handle custom JSON body for HTTP API with OSB 12.2.1
categories: OSB tips
tags: [OSB, JSON, API, Web services, Javascript]
authors:
- cliops
- denzza
---
In this post, we will learn how to handle custom JSON messages for HTTP services with OSB 12.2.1, there we go:

### INTRODUCTION ###

The REST API connector allows to use REST web services via HTTP but sometimes we need to struggle when the JSON data structure is not compatible with XML structure at all.

For solving this, is necessary to know what is the complete structure of the request REST service, since the structure of the content body could be a bit tricky to manage then we are going to do some steps to handle it in a good way.


### SOLUTION ###

Let's start creating the HTTP transport as the image below


![](/images/2021-04-13-Handle-Custom-JSON-Http-Message-with-OSB-12C/image1.png)

Then select the request message type as TEXT, so you will be able to send the message body as TEXT with any type structure.

![](/images/2021-04-13-Handle-Custom-JSON-Http-Message-with-OSB-12C/image2.png)

Next step is to build the JSON request as string, we can use xquery function as the image below

![](/images/2021-04-13-Handle-Custom-JSON-Http-Message-with-OSB-12C/image4.png)

As you can see, it is possible to build any JSON body since we start creating from scratch as string.

When we have the request complete as string the next step is call the rest service and set is as JSON using transport header.

![](/images/2021-04-13-Handle-Custom-JSON-Http-Message-with-OSB-12C/image3.png)

Finally use the xquery mapping file in a assign component and include this as a request of the service callout
![](/images/2021-04-13-Handle-Custom-JSON-Http-Message-with-OSB-12C/image5.png)


If we want to handle the request or response as Json and not as String in the pipeline then there is a good feature added in OSB 12.2.1 called Javascript located in the Message Processing section.

![](/images/2021-04-13-Handle-Custom-JSON-Http-Message-with-OSB-12C/image6.png)
Basically, this new component allow us to insert javascript code where we can create or modify existing variables in the pipeline. For example if we want to get the value of a field in the Json response, then we initiate with the variable called "process" and reference the response variable. In this case the variable of the response is a JSON object so I pick up the id field which is number and parse it as string.


### Conclusion ###

There are several solutions to achieve this solution, like for example using java callout. So this workaround that I provide fits more with the new feature of 12.2.1 (javascript) and is not complex at all to complete the integration.


If you have any questions let me know, thanks.
