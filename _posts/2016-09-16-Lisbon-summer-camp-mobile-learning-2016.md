---
layout: post
title: Lisbon Summer camp 2016 Mobile training
categories: Mobile tips
tags: [MAX, Oracle forms, MCS, Mobile]
author: cliops
---
In this post, we will see an overview about all the knowledge learned in the Lisbon summer camp - Mobile section. There we go:

### Building AuraPlayer web Services ###

First, we created REST web services to manage the Forms summit application using AuraPlayer's Service Manager. We used IE to launch the summit Forms.

![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/AuraPlayerServiceManager.jpg)

We created several services, pushing the button "record services.

![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/AuraPlayer-createService.jpg)

Then we defined the service name an the form URL. The URL form is connected to a Forms Server where AuraPlayer is installed. We have some options that we can use on AuraPlayer Manager in the record time.

![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/AuraPlayer-optionsRecordTime.jpg)

Also, we handled a "Oracle Forms" window that was opened automatically on IE.

![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/AuraPlayer-OracleForms.jpg)

Every time that we wanted to save and stop the recording, we pushed Save & Exit.
![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/AuraPlayer-saveAndExit.jpg)

To test the services, from the "Service Details" page, we used the "Test" button
![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/AuraPlayer-testButton.jpg)

Then the Service Tester page is displayed, The input parameters frame displays the values that would be sent to the service in this test.

The result could be displayed as XML or JSON if the "Show as Json chekbox" was checked.

### Building MCS CUSTOM API ###

We accesed to Mobile cloud service page and we a went to Applications -> MobileBack Ends. Then we created a new backend.

![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-Backend.jpg)

Then we created a new Mobile BackEnd.

![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-createBackEnd.jpg)

The details of the new Backend are displayed and we continued to design it.

![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-createBackEnd-2.jpg)


We added new resources like apis and connectors, I will show bellow:

 - Create custom APIs:

 Select APIs

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-APIS.jpg)

 Then New API, we entered the API name and a short description.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-APIDetails.jpg)

  On the security option set the "Login Required" to "OFF".

  ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-APIsecurity.jpg)

  On the Endpoints option we created new resources, there we could define the path, type of the Method and the description of the endpoint.

  ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-APIResources.jpg)

  For each resource we could define the Request, Responses and Media Types.

  ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-APISMethods.jpg)

  To check the validation of the json sample data we used an online tool like: http://www.jsoneditoronline.org/

  At the end of the session we created several resources as the image bellow:

  ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-APIlistResources.jpg)

  On the schema option we created schemas to relate with the json content handled by the REST services.

  ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-APISchemas.jpg)

 -  Create REST connector

 Select connectors

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-Connectors.jpg)

 Then we defined the Api name and the Remote URL that we created using AuraPlayer manager.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-ConnectorsDetails.jpg)

 We could test the created services using the endpoint URLs.
 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-ConnectorsTest.jpg)

 Then we modified the implementation of the APIS to use the connectors created previously. In the API section we went to the Implementation option and downloaded the API implementation pressing the Javascript scaffold green button in the center.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-APIimplementation.jpg)

 The archive file is downloaded in the computer and it contained 4 files, package.json, ReadMe.md, samples.txt and summitapplicationapi_00.js.

  - package.json includes metadata about the API.
  - ReadMe.md includes explanation about the other files.
  - Samples.txt contains a selection of Oracle MCS custom code examples
  - summitapplicationapi_00.js is the main javascript file. The functions included in the javascript file define where we are expecting API implementation code to appear.

  package.json

  ```javascript
  {
  "name" : "summitapplicationapi_13",
  "version" : "1.0.0",
  "description" : "Summit Application resources for PaaS4SaaS Summer Camp 2016",
  "main" : "summitapplicationapi_13.js",
  "oracleMobile" : {
    "dependencies" : {
      "apis" : { },
      "connectors" : { "/mobile/connector/SummitAuraplayerServices_13":"1.0" }
       }
      }
   }
   ```

  Here we could define and handle the json request or response of the connector while the API rest services are invoked.
 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-APISimplementationJson.jpg)

 Finally, repackage the archive, upload it to MCS, test in MCS UI and publish.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-APISimplementationTesting.jpg)

### Create a mobile application with Oracle MAX ###

 Go back to MCS Applications and select mobile Applications.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-MobileApplication.jpg)

 We created a new application in the Mobile Application Accelerator (MAX)

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-MAX.jpg)

 The steps to design the mobile application was really easy. Basically, step by step we created the view dragged and dropped some components like views, lists forms.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-MAXCreateViews.jpg)

 Also we added some charts

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-MAXCharts.jpg)

 To include data, first we searched for the API that we published and we selected the version that we wanted to use.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-MAXAPI.jpg)

 Then the fields included in the json schema of the custom API are displayed.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-MAXDATA.jpg)

 Before to deploy in a smartphone, we tested the MAX Application in the MAX simulator. So we clicked on the run icon.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-MAXRunSimulator.jpg)

 We can test the simulator for Android and IOS on different screen sizes.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-MAXSimulator.jpg)

 Then when everything was going fine, we tested on the device, so we clicked on the button "Test on Phone".

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-MAXTestPhone.jpg)

 It gave a QR code generated to use with another application called Oracle Mobile Application Accelerator. We could find in Apple store and Google Play store.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-MAXQR.jpg)

 Once installed and running, we selected the plus icon at the bottom right of the MAX app, this started the QR code scanner. Then we scanned the QR code generated by the app.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-MAXQRScanner.jpg)

 The application was automatically starting after installation. Once the app was closed and we wanted to run it again, we could see that app in the MAX application container.

 ![](/images/2016-09-16-Lisbon-summer-camp-mobile-learning-2016/MCS-MAXDeployedApp.jpg)
