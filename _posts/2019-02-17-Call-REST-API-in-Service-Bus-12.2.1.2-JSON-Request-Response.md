---
layout: post
title: Call REST API in Service Bus 12.2.1.2 (JSON Request/Response)
categories: osb JavaScript json REST
tags: [osb, JavaScript, json, REST]
author: denzza
---

In this example we will create similar integration as in [previous post](http://blog.sysco.no/osb/javascript/json/Retrieve-data-from-JSON-Using-JavaScript-Expressions-in-Oracle-Service-Bus-12c/) and only difference will be that both Request and Response will be in JSON format. So let’s call it a typed REST service. We will call a REST API with JSON data format (parameters as input) and as a Response we will get JSON also. So, if you have a requirement to work with some RESTful API that works only with JSON data format you can do it easy in the Service Bus 12.2.1.2 version without converting to XSD. All data mapping can be done in JavaScript activity. JavaScript is not limited to REST services. We can use JavaScript in any service.
Here we will call Weather data Rest API and retrieve some JSON information, and this is an example URL of this API:

<http://api.openweathermap.org/data/2.5/weather?q=Oslo&appid=77d7bf10384d0beb90e02f4533c37535>

For more information on this API and how to use it you can visit [Open Wheater site](http://openweathermap.org/current)

## Demo

We will create:
- REST Proxy Service – where we will provide the parameter for the API call
- Pipeline 
- REST Business service – actual call to the REST API.

This is how end result should look like and what folder structure is proposed.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/6_ProjectFolderStructure.jpg)

As a first step after creating the project start by creating the necessary folders: Business, Proxy, Pipeline and Resources.

## REST Proxy Service

Right click in the Proxy Service lane and choose REST, or another option is to drag and drop the REST activity from the Service Bus Components from the right side of the jDeveloper.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/1_RESTAdapter.jpg)

Now name your Proxy service which will be used for posting the JSON parameters as a request.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/1_RESTAdapter_2.jpg)

In the next step in the Methods section click on the plus icon which represents “Add a new REST method”.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/1_RESTAdapter_3.jpg)

And a new window will show up for Request and Response methods. First, we will create a Request method by adding the parameters. In the section URI Parameters click on the green plus sign to add a new parameter and fill out the info as in the picture. You don’t need to fill the Description field, that was just a short explanation.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/1_RESTAdapter_4.jpg)

Now do the same for second parameter:

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/1_RESTAdapter_5.jpg)

Request tab should look like next picture:

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/1_RESTAdapter_6.jpg)

For Response tab we can choose both JSON and XML as a payload options: 

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/1_RESTAdapter_7.jpg)

Click Finish and your Proxy service will be created and one important thing will be created also and that is WADL. If you created folder named Resources generated WADL will be stored in that folder automatically.

## Pipeline creation

Next, we will create a pipeline by right clicking in the Pipeline/Split Joins lane. 

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/2_PipelineCreation_1.jpg)

Now name your pipeline and choose the folder where you want it to be created. 

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/2_PipelineCreation.jpg)

In the next wizard’s window we will do 3 steps:
1. Uncheck the “Expose as a Proxy Service” – we already have a proxy service so we will not use this feature.
2. Select REST as a Service Type and
3. Click on the “Browse WADLs” icon to select a WADL.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/2_PipelineCreation_2.jpg)

Here you will select the WADL created from our Proxy service. And as a last step click finish and you Pipeline will be created. Now just what is left is to connect the Proxy Service with the Pipeline.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/2_PipelineCreation_3.jpg) ![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/2_PipelineCreation_4.jpg) 


## REST Business Service

Next is to set REST Business Service which will do the call to the Weather REST API. Right click in the External Services lane and choose REST again. Name your Business Service and click Next.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/3_RestBusinessService_1.jpg)

In next window we have 3 steps to do:
1. Add Base URI from the Weather API we will use: <http://api.openweathermap.org/data>
2. Then you need to add and create a REST resource by clicking the green plus icon in the Resources section.
3. In Relative path add: 2.5/weather – and you can see the full path and double check that it’s ok.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/3_RestBusinessService.jpg)

Next step is to add a REST method by clicking the green plus icon in the Methods section.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/3_RestBusinessService_0.jpg)

For Request do next:
1. Name your method.
2. Select the HTTP verb as GET.
3. Repeat the steps for parameters as in the creation of Proxy Service. 

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/3_RestBusinessService_2.jpg)

Response tab can look like this using both JSON and XML. Click Finish.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/3_RestBusinessService_3.jpg)

Final version of the REST Business service and how it should look like. Also Design view where you connect Business service with Pipeline.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/3_RestBusinessService_4.jpg)

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/4_DesignView.jpg)


## Pipeline implementation of the REST call

Now we will implement the logic behind the REST API call in the pipeline. So, do a double click to open the pipeline. This is the image you should have once inside the pipeline. First thing to do is to click on the Routing and in the bottom in Routing Properties change the Operation to “GetWeatherData” in our case which we created in Business service. 

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/5_PipelineImplementation_1.jpg)

To call a REST service with parameters we have to use a Query parameters inside the transport request element of the call towards our Weather Data API. So, we need to add them to the **$outbound** variable. This is the same way how we receive the query parameters from **$inbound** variable in the request. We will do that by drag/dropping the **Insert activity** from the Components section on the right side into the Request Action lane.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/5_PipelineImplementation_2.jpg)

Set the Insert Properties like this:
Value:

```xml
<http:query-parameters>
    <http:parameter name="q" value="{$inbound/ctx:transport/ctx:request/http:query-parameters/http:parameter[@name ='cityName']/@value}"/>
    <http:parameter name="appid" value="{$inbound/ctx:transport/ctx:request/http:query-parameters/http:parameter[@name ='appID']/@value}"/>
</http:query-parameters>
```

Position: as first child of
Location: select outbound variable from the drop down and add expression for the specific element in this case we need: 

```shell
./ctx:transport/ctx:request
```

This is all we need to call and get the Weather data and get a JSON Response back from the API. Now we can deploy and test the integration. 

## Testing

Log to your service bus console and find the deployed project. We can test this in two ways using a “Test Console”. We can do it from the Proxy service directly but for Debugging and detailed view of the execution flow we will use Pipeline.

So, expand your Pipeline folder and click on the pipeline name to open it. When it’s opened click on the green play button on the right side of the screen inside the pipeline window. 

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/8_Testing.jpg)

In the Test Console fill the necessary fields for testing and just click **Execute**.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/8_Testing_2.jpg)

Now you should get the results:

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/8_Testing_3.jpg)

Next what we can try and test is to extract some of the data from this JSON response and create a small XML for example. This will be done by adding the JavaScript activity in Response Activity line inside the Pipeline.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/9_JSON_JavaScript_XML_0.jpg)

Now we need to populate the JavaScript Value. 

And you can use this code for creating new variable in JavaScript activity.

```xml
process.XMLResponse = 
'<weather-data>\n'+
  '\t<temperature>' + process.body.main.temp + '</temperature>\n'+
  '\t<main>' + process.body.weather[0].main + '</main>\n'+
  '\t<description>' + process.body.weather[0].description + '</description>\n'+
'</weather-data>'
```

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/9_JSON_JavaScript_XML.jpg)

Click ok, save your work in the pipeline and redeploy the integration so we can test it again.

Just repeat the same steps from the first test and go to the Invocation trace section and expand the **$outbound** and our **$XMLResponse** variable.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/8_Testing_4.jpg)

In the **$outbound** variable we can see the ```<query-parameters>``` element with our parameters values towards the Weather API. 

And **$body** variable is clean JSON data without any conversions.

![](/images/2019-02-17-Call-REST-API-in-Service-Bus-12.2.1.2-JSON-Request-Response/8_Testing_5.jpg)
