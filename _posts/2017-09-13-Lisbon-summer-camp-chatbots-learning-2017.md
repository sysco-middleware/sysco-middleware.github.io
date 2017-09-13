---
layout: post
title: Lisbon Summer camp 2017 Chatbots & Oracle JET training
categories: Mobile tips
tags: [Chatbots, MCS, Oracle JET]
author: cliops
---
In this post, we will see an overview about all the knowledge learned in the Lisbon summer camp 2017- Intelligents chatbots and Oracle JET section. There we go:

### Explore a ChatBot ###

First, we needed to access to the Oracle Cloud Bot Service provided in the training.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image1.jpg)

We are going to explore the bot called "MasterBot".

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image2.jpg)

Then on the left we can see a vertical list of icons allowing us to work on the ChatBot's intents, entities, flow, components and settings.

Intents, are used to keep utterances that a user can send in a message according to an specific context. In this case, the bot can find 3 intents related to Balances, Send Money and Track Spending. In the image below there are several utterances that can be used. For example, How much money do I have in all my accounts.
![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image3.jpg)

Also, every intent could be related to several entities so every utterance could have multiple options to find a match by the ChatBot.

Entities, are used to map every word or group of words that are considered important in the utterance.
For example, in the image above there is a entity called AccountType, so the possible values related to it would be  checking, savings and credit card. In addition, is possible to add synonyms to every value so it will increase the chance to find the intent. Finally, there are several common entities that could be used like DATE, TIME, EMAIL and others.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image4.jpg)

The entities are assigned in the Intents section.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image5.jpg)

Flows, in this section we will define the behavior of the ChatBot written in YAML code.
By default, the ChatBot looks like the image below.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image6.jpg)

In the training we explored a flow already developed, and as we can see in the image below there is a segmentation for every piece of code like Header, Context variables, Intent States, Start Balance, Ask Balance Account Type, Print Balance and Unresolved. In this version of the ChatBot 1.0 we learned to code in this way but in the next version there will be a nice GUI to avoid writing code in this way, but it was good to know how it works.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image7.jpg)

So basically, it will request for your balance and print an amount only if the utterance finds a match with the intent "Balance". Otherwise, it will print "Unable to resolve intent!".

Custom components, are components used from rest services that bot developers create and deploy onto any infrastructure they want (as long as they are exposed on the internet), and then configure bots to call them.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image8.jpg)

In this case, the rest services were implemented using Oracle MCS

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image9.jpg)

Settings, are used to set channels for facebook or webhook, and services to import custom components mentioned previously.

 ![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image10.jpg)


### Test a ChatBot ###

We can test the ChatBot clicking on the "Play" button. Also you can validate the YAML code clicking in the "Validate" button, and train the bot if is necessary according to the messages in the dialog.

 ![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image11.jpg)

To examine how the train works, we click in Intent tab, so after sending a message the bot will return a list of all the intents and a confidence percentage for each as the image below.

 ![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image12.jpg)


### Integrate ChatBot with Facebook Messenger ###

In this section, we published the MasterBot through Facebook Messenger, such as users can access the intelligent bot. To do this we created an intelligent Bot Facebook Messenger Channel to hold  the settings required for Facebook Messenger and Intelligent Bot to connect each other.

First we create a personal facebook page

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image13.jpg)

Then we enable our account as Facebook developer account. Going to  https://developers.facebook.com

Create an APP ID, the app will represent the chatbot in Facebook.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image14.jpg)

Selecting Dashboard menu option shows the top level information about the application of which App Secret key will be needed.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image16.jpg)

Then enable the messenger product and webhook for the page we just created.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image17.jpg)

In the resulting messenger settings, we needed to configure the facebook page we created for Messenger. So in the Token Generation section we drop down to select our page and doing this will generate a Page Access Token.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image18.jpg)

Then we configured the Intelligent Bot Facebook Messenger channel in the settings menu.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image19.jpg)

In the create channel dialog we set the channel type Facebook Messenger and set the Access Token and App Secret Key provided by Facebook developer.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image20.jpg)

After this channel was created successfully, we copied down the Verify token and Webhook URL. We will use these on Facebook side to hook the ChatBot to our app.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image21.jpg)

Returning to the Facebook App, in the resulting new page subscription dialog for the Callback URL, we wrote down the Verify Token and Webhook URL. For subscription fields we set messages and messaging_postback.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image22.jpg)

In the Webhooks section we selected the name of the page and then click on the subscribe button.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image23.jpg)

Finally we tested the ChatBot.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image24.jpg)

### Integrate ChatBot with Oracle Jet ###

First, we needed to prepare the environment to use Oracle Jet that is a modular kit and open source framewrok that contains javaScript libraries along with a set of Oracle contributed JavaScript libraries that make it simple to build applications that consume and interact with Oracle products and services, especially Oracle Cloud Services.

We followed some simple steps from this side to prepare the environment https://netbeans.org/kb/docs/webclient/ojet-settingup.html

Also it provides a usefull CookBook where we can just copy and paste some nice samples to learn easily from this side http://www.oracle.com/webfolder/technetwork/jet/jetCookbook.html

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image27.jpg)

Following the steps to prepare the environment it will create a sample project where we can see the arquitecture of the source files.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image26.jpg)

We modified in the src folder and included some sources code provided in the session.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image29.jpg)

Then we modified the Incidents.js file as the source below.


```javascript
define(['ojs/ojcore', 'knockout', 'jquery','jet-composites/bot-client/loader'],
function(oj, ko, $) {

  function IncidentsViewModel() {
    var self = this;
    self.websocketConnectionUrl = 'ws://129.157.177.23:3000/chat/ws';
    self.userId = '5c82a414-e2d0-45fd-b6a2-8ca3b9c09160';
    self.channel = '10FA3026-5CAC-4ADE-AFD7-5678003240CD';

    self.reset = function() {
          var ccaNode = $('#bot-client');
          ccaNode[0].reset();              
    }

```
Finally we tested in the web browser locally.

![](/images/2017-09-13-Lisbon-summer-camp-chatbots-learning-2017/Image30.jpg)

### Conclusion ###

The summer camp was absolutely useful to understand how a ChatBot works and could interact with several technologies. Also the chatbot seems that will replace several mobile applications that provide particular services, so it would not be necessary to install a mobile application to request a specific service, a chatbot could do it for you.  About Oracle Jet, I would say is a great framework to build web applications easily because there is a wonderful cookbook where we can just copy and apply some changes in our own projects. 
