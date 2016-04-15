---
layout: post
title: Using Queues with Delivery Failure
categories: Weblogic JMS
tags: [Java Message Service, JMS, Error Destination, Redelivery Limit, Weblogic]
author: raul
---

## Using JMX to Monitor Oracle Weblogic Partitions ##

Some days ago I realized that when a message is created through the Admin Console the Queue parameters set to use an Error Destination do not override the message’s parameters, it made me lose time because I thought that my application had a problem. However, the problem was caused because the “Redelivery Limit” set on the message by the Admin Console was no overwritten by the “Redelivery Limit” set in the Queue.

You can read the guide using the following link.

[***JMS, Error Destination and Redelivery Limit***](/files/guides/JMSErrorDestinationRedeliveryLimit.pdf)