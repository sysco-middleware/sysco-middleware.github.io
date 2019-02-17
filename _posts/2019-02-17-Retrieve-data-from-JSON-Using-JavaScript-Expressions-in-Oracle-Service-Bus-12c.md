---
layout: post
title: Retrieve data from JSON Using JavaScript expressions in Oracle Service Bus 12.2.1.2
categories: osb JavaScript json
tags: [osb, JavaScript, json]
author: denzza
---

# Introduction

SOA Suite with Service Bus 12.2.1.2 provides a JavaScript action, which can be used in OSB pipelines. JavaScript action allows you to include snippets of JavaScript code to be executed during pipeline processing.

The most common case for using JavaScript is when dealing with JSON objects in REST services. Now you can use JSON object directly by using the JavaScript activity. So rather than converting the payload to XML and using xQuery or XSLT for mapping the data here we can use JavaScript directly to manipulate the JSON object. The JavaScript engine used in Service Bus also allows you to easily reference XML elements, making it easier to handle both JSON and XML-style payloads in JavaScript.

TIP: The JavaScript action is available for any pipeline type, not just Native REST pipelines.

Service Bus binds a globally-scoped object called **process**

To access a $body variable in the Service Bus message context, use an expression like the following:

**process.body**

To create a variable in the Service Bus message context, use an expression like the following:

**process.newVar = …;**

To delete a variable, use the JavaScript delete operator:

**delete process.var;**

To access a variable, use below JavaScript:

**process.firstName = process.body.employees.firstName;**

The following expression returns the value of the inbound HTTP Content-Type header:

**process.inbound.ctx::transport.ctx::request.tp::headers.http::["Content-Type"].text()**

# Demo
In this demo example I will demonstrate how to use JSON as a Request and emphasis on fetching the JSON payload using JavaScript activity.

First we will create a simple HTTP Transport with REST as a request service which we can do like in the next pictures following the create wizard. 

![](/images/2019-02-17-Retrieve-data-from-JSON-Using-JavaScript-Expressions-in-Oracle-Service-Bus-12c/01_RestTransport.jpg)

![](/images/2019-02-17-Retrieve-data-from-JSON-Using-JavaScript-Expressions-in-Oracle-Service-Bus-12c/02_RestProxyCreate.jpg)

![](/images/2019-02-17-Retrieve-data-from-JSON-Using-JavaScript-Expressions-in-Oracle-Service-Bus-12c/03_RestProxyCreate.jpg)

![](/images/2019-02-17-Retrieve-data-from-JSON-Using-JavaScript-Expressions-in-Oracle-Service-Bus-12c/04_RestProxyCreate.jpg)

And this should be result in the project Design view.

![](/images/2019-02-17-Retrieve-data-from-JSON-Using-JavaScript-Expressions-in-Oracle-Service-Bus-12c/04_RestProxyFinal.jpg)

Now we will open the pipeline view and create Pipeline Pair in which we have Request and Response pipelines. In the Request pipeline we will use JavaScript activity to retrieve the JSON payload data. And in Response pipeline a Replace activity for mapping the JSON data to an XML message as an example.

![](/images/2019-02-17-Retrieve-data-from-JSON-Using-JavaScript-Expressions-in-Oracle-Service-Bus-12c/05_PipelineDev.jpg)

Below is our JSON Request Payload:

```json
{
 "employees": {
  "firstName": "John",
  "lastName": "Doe"
 }
}
```

For Retrieving **firstName** and **lastName** from JSON Request, below is JavaScript Expressions in the Request Stage:

![](/images/2019-02-17-Retrieve-data-from-JSON-Using-JavaScript-Expressions-in-Oracle-Service-Bus-12c/06_JavaScript.jpg)

In the Response Stage, in a replace action we will create a simple mapping for an XML Response. 

![](/images/2019-02-17-Retrieve-data-from-JSON-Using-JavaScript-Expressions-in-Oracle-Service-Bus-12c/07_ReplaceXML.jpg)


# Testing 

And now we are set to deploy our integration and do a first test. 
In the Service Bus Test console select Media Type: application/json and paste the JSON payload as in picture below.

![](/images/2019-02-17-Retrieve-data-from-JSON-Using-JavaScript-Expressions-in-Oracle-Service-Bus-12c/08_TestingPayload.jpg)

This should be a result of our test:

![](/images/2019-02-17-Retrieve-data-from-JSON-Using-JavaScript-Expressions-in-Oracle-Service-Bus-12c/09_TestResultRESTJSONJavaScript.jpg)
