---
layout: post
title: Extending Sesam integration platform with custom data sink service. 
categories: Sesam Integration
tags: [Sesam, Sesam.io, Azure Service Bus]
author: TimurSamkharadze
---

## Introduction

Sesam is an integration platform. As any other integration platform, Sesam has a lot of connectors to read data from different sources which could be databases, API's, streaming platforms such as Apache Kafka, or to send data to them.  
Today we are going to write a simple connector to push data from Sesam datahub to Azure Service Bus. Sesam platform uses Docker to run extensions and every extension is actually a REST service which takes array of JSON objects as input (data sink service), or returns array of JSON objects (data source service), or do both in case of data transformation service. 

If you want to test it for youself, you will need to do 3 things:
* Obtain Sesam account. We explained in [this post](http://blog.sysco.no/data/analysis/Making-data-pipelines-with-Sesam-and-Oslo-City-Bike-public-API-in-5-minutes/) how to do that
* Obtain Azure account [here](https://azure.microsoft.com/en-us/free/)
* Install Docker

We are going to use Python in this post, but you may use programming language of your choice.

## Let's start
We are going to run Azure Service bus sink service in isolated environment, so we don't need to deal with such things as authentication. In such case all required information needed for service configuration must be provided through environmental variables. You may also use [Sesam vault](https://docs.sesam.io/security.html#secrets-manager) known as _Sesam secret manager_ to provide for example credentials without exposing them.

To connect to Azure Service Bus using SAS authentication schema we will need:  
```python
SERVICE_NAMESPACE       = os.environ.get("SERVICE_NAMESPACE")       # Azure Service Bus service namespace
SERVICE_SAS_TOKEN_NAME  = os.environ.get("SERVICE_SAS_TOKEN_NAME")  # SAS key name
SERVICE_SAS_TOKEN_VALUE = os.environ.get("SERVICE_SAS_TOKEN_VALUE") # SAS key
```

And to implement a REST service that can communicate with Azure Service Bus in Python we need also 3 things
* Azure Python library
* Application server
* Framework for creating web applications  

All these 3 requirements may be simply satisfied by placing them to file _requirements.txt_ and instructing Python packet manager to install them. This will be done later when we build docker image and now you only need to store this file in your project folder.

```
azure-servicebus==0.50.0
waitress==1.2.1
flask==1.0.2
```

Now it's time to implement our service. This service doesn't contain any built in security or authentication providers and is suitable to run only in isolated environment - inside of Sesam node.


```python
import os
import logging
import json

from azure.servicebus.control_client import ServiceBusService, Message
from waitress import serve
from flask import Flask, Response, request, abort

APP = Flask(__name__)

log_level = logging.getLevelName(os.environ.get("LOG_LEVEL", "INFO"))
logging.basicConfig(level=log_level)

SERVICE_NAMESPACE = os.environ.get("SERVICE_NAMESPACE")
SERVICE_SAS_TOKEN_NAME = os.environ.get("SERVICE_SAS_TOKEN_NAME")
SERVICE_SAS_TOKEN_VALUE = os.environ.get("SERVICE_SAS_TOKEN_VALUE")
PAYLOAD_KEY = os.environ.get("PAYLOAD_KEY")


@APP.route("/<queue_name>", methods=['POST'])
def process_request(queue_name):
    """
    Endpoint to publish messages to Azure service bus
    :param queue_name: name of queue to publish messages to
    :return:
    """
    input_data = request.get_json()
    bus_service = ServiceBusService(service_namespace=SERVICE_NAMESPACE,
                                    shared_access_key_name=SERVICE_SAS_TOKEN_NAME,
                                    shared_access_key_value=SERVICE_SAS_TOKEN_VALUE)

    for index, input_entity in enumerate(input_data):
        data: str = json.dumps(input_entity[PAYLOAD_KEY] if PAYLOAD_KEY else input_entity).encode(
            "utf-8")
        msg = Message(data)
        try:
            bus_service.send_queue_message(queue_name, msg)
            logging.info("Entity %s sent successfully", input_entity["_id"])
        except Exception as e:
            logging.error(e)
            abort(500)
    return Response()


if __name__ == "__main__":
    """
    Program entry point
    """
    port = os.environ.get('PORT', 5000)
    logging.info("starting service on port %d", port)
    serve(APP, host='0.0.0.0', port=port)
```
    
This service has one endpoint that serves POST requests and takes queue name as path parameter. Payload must be an array of JSON objects. You may also assign and use environmental variable _PAYLOAD\_KEY_ if you don't need to send a whole Sesam entity with its metadata but only a part of it.

Next thing we need to do to complete our service is to create Docker image of it and push it to a Docker repository where Sesam will be able to pull it. I'm going to use public Dockerhub repository, but images may be placed in any docker repository including private repositories.  

Add Dockerfile to your project and build & push image.

```
FROM python:3-slim

COPY ./* /service/
WORKDIR /service

RUN pip install -r requirements.txt

EXPOSE 5000/tcp

ENTRYPOINT ["python3"]
CMD ["service.py"]
```

Now it's time to use our newly created service to send some data from Sesam to Azure Service Bus.

Login in into Sesam portal and join free of charge "Open Sesam" subscription (this subscription is for open data so don't use it in production or with sensitive data).
Press "Systems" and create new system using following system definition.
```json
{
  "_id": "azure-sb-sink",
  "type": "system:microservice",
  "docker": {
    "environment": {
      "SERVICE_NAMESPACE": "$SECRET(service_namespace)",
      "SERVICE_SAS_TOKEN_NAME": "$SECRET(service_sas_token_name)",
      "SERVICE_SAS_TOKEN_VALUE": "$SECRET(service_sas_token_value)"
    },
    "image": "<location of your image>",
    "port": 5000
  },
  "verify_ssl": true
}

```
Press "Save" - you system is stored. Then open "Secrets" tab and add following items to Sesam vault:
* service_namespace - with your Azure Service Bus namespace
* service_sas_token_name - with SAS token name
* service_sas_token_value - with content of SAS token

Now your system will be able to use them without exposing as plain text to everyone including yourself.

We have now a data sink system and next step is to send some data to it. Open "Pipes" menu and create new pipe using following pipe definition: 
```json
{
  "_id": "azure-sb-sink-pipe",
  "type": "pipe",
  "source": {
    "type": "embedded",
    "entities": [{
      "_id": "1",
      "value": "foo"
    }, {
      "_id": "2",
      "value": "bar"
    }]
  },
  "sink": {
    "type": "json",
    "system": "azure-sb-sink",
    "url": "my_queue"
  },
  "pump": {
    "schedule_interval": 60
  }
}
```
We will use embedded data set with 2 simple entities.  
That's all! Now this pipe will send its embedded data to Azure Service Bus every 60 seconds.  
In this post we described how to extend functionality of Sesam integration platform by adding a custom data sink connector with about 50 lines of code.  
