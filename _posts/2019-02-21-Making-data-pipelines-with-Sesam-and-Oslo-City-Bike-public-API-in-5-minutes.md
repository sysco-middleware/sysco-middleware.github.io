---
layout: post
title: Making data pipelines with Sesam and Oslo City Bike public API in 5 minutes. 
categories: Data analysis
tags: [Sesam, Sesam.io, Oslo bysikkel]
author: TimurSamkharadze
---

# Introduction

In this very short post we are going to create a simple pipeline for getting data from Oslo City Bike public API service
using Sesam.io integration platform with 5-10 minutes of our time.   We don't need so much time here because it's easy
to make integrations with Sesam. You don't need to write programs or deal with infrastructure to deploy them.  
Data we are going to collect will provide us bike availability at different stations over time.  

Oslo city bike is a simplest way to explore Oslo by using public bicycle. There are more than 200 bike stations in Oslo
and more than 50 000 users of this service.  

Sesam.io is a integration platform delivered as PAAS. It is optimised for collecting or receiving data from 
source systems, transforming data, and providing data for target systems.

# Let's begin
First of all we need to create Sesam user account [here](https://portal.sesam.io) and join "Open Sesam" subscription which is free of charge.  

![](/images/2019-02-21-Making-data-pipelines-with-Sesam-and-Oslo-City-Bike-public-API-in-5-minutes/02_sesam_studio.png)  

So we need a second one account, this time for [Oslo City Bike service](https://developer.oslobysykkel.no) and we need to   
create new API client. This procedure is very easy and we will get API key needed for making API requests.   

![](/images/2019-02-21-Making-data-pipelines-with-Sesam-and-Oslo-City-Bike-public-API-in-5-minutes/01_osb_client.png)  

After we completed two preparation steps we can start to do something useful.

Sesam operates with definitions "Systems" and "Pipes" where System may be any data source or data sink such as databases, REST API's, cloud services, files on network shares, etc. 
A pipe represents a data flow from source to sink and specifies 
how data entities transformed, enriched or filtered on their way.  

Let's create our first system that will represent Oslo City Bike API.  
In Sesam management studio dashboard click on "Systems" menu and then "New system" button.
System definition is a JSON object where we need to provide free-chosen id, type - system:url, add authentication headers  
which will be sent with every request and assign URL pattern for API. That's all.
```json
{
  "_id": "oslo-bysikkel",
  "type": "system:url",
  "headers": {
    "Client-Identifier": "$SECRET(access-token)"
  },
  "url_pattern": "https://oslobysykkel.no/api/v1/%s",
  "verify_ssl": true
}
```
Press save to store new system. You may see that Client-Identifier header which obviously used to send Oslo City Bike API key
contains strange value starting with '$SECRET'. 
It means Sesam will try to find 'access-token' attribute in Sesam vault.
Sesam vault is write only encrypted storage for sensitive data like application credentials. We are going now to add our
API key for Oslo City Bike API to this vault. Simply click on "Secrets" tab and "Add secret" button after that.  
![](/images/2019-02-21-Making-data-pipelines-with-Sesam-and-Oslo-City-Bike-public-API-in-5-minutes/03_sesam_studio.png)  
Assign name "access-token" and paste your Oslo City Bike API key into value field. Press "Add". That's all, now your key
is securely stored in Sesam vault. 
After creating data source system we need to create some pipes which will fetch data from that system. In Sesam dashboard click on "Pipes" menu and then "New Pipe" button.  
 
Pipe configuration is also simple JSON object which has 4 main components:
* Source - which system is our data source
* Transform - which transformations will be applied to every data entity flowing through this pipe
* Sink - target system, if omitted then pipe will produce a data set with the same name as pipe id
* Pump - contains "pumping" options such as pipe schedule, dead letter queue options, etc

Our first pipe will fetch data from "/stations" Oslo City Bike API endpoint that provides data for all stations currently in operation.  
```json
{
  "_id": "oslo-bysikkel-stations",
  "type": "pipe",
  "source": {
    "type": "json",
    "system": "oslo-bysikkel",
    "url": "/stations"
  },
  "transform": {
    "type": "dtl",
    "rules": {
      "default": [
        ["create",
          ["apply", "add-id", "_S.stations"]
        ]
      ],
      "add-id": [
        ["add", "_id",
          ["string", "_S.id"]
        ],
        ["copy", "*"]
      ]
    }
  },
  "pump": {
    "cron_expression": "0 0 1 * ?"
  }
}
```
Here we use our previously created system as source of data, schedule pipe to run monthly (stations are quite static and
we don't expect many changes here) and apply some cryptic transformation that is needed to be explained.  
Sesam pipe expect to get an array of data entities on its input while Oslo City Bike API returns a JSON object. So we need
to use built in [Data Transformation Language](https://docs.sesam.io/DTLReferenceGuide.html) (or simply DTL) to transform input object into array of stations and emit every station object as own entity. Second thing we do here is "id" assign.  
Every entity in Sesam must have an string attribute named "\_id" - entity key. It's used for entity identification, compaction, etc.
That's all, press "Save" and then "Start" buttons. After a second a new data set will be populated with stations data.  

Now let's create a second pipe which will fetch information about station availability at current time. Pipe setup is almost the same and we can simply duplicate our first pipe by clicking respective link in pipe options.  

![](/images/2019-02-21-Making-data-pipelines-with-Sesam-and-Oslo-City-Bike-public-API-in-5-minutes/04_sesam_studio.png)  


Now, let's do some changes.
* Change pipe id to desired
* Change source URL to "/stations/availability" endpoint
* In transformation section add "updated\_at" timestamp from input data root to every entity
* Add timestamp to entity id to unique identify every station and time point we collect.
* Add station name from our first data set by performing join operation (DTL function "hops")
* Change pipe pump schedule to fixed interval (900 seconds or 15 minutes in our example)
* Add explicit sink section with attribute "deletion\_tracking" equal to false  

We need deletion\_tracking disabled because by default full data set synchronization will delete previous entities.
Sesam, however, has functionality for incremental data sync as well as for full sync.  
```json
{
  "_id": "oslo-bysikkel-availability",
  "type": "pipe",
  "source": {
    "type": "json",
    "system": "oslo-bysikkel",
    "url": "/stations/availability"
  },
  "sink": {
    "type": "dataset",
    "dataset": "oslo-bysikkel-availability",
    "deletion_tracking": false
  },
  "transform": [{
    "type": "dtl",
    "rules": {
      "default": [
        ["create",
          ["apply", "add-id", "_S.stations"]
        ]
      ],
      "add-id": [
        ["add", "_id",
          ["concat",
            ["string", "_S.id"], "-", "_P._S.updated_at"]
        ],
        ["copy", "*"],
        ["add", "updated_at", "_P._S.updated_at"]
      ]
    }
  }, {
    "type": "dtl",
    "rules": {
      "default": [
        ["copy", "*"],
        ["add", "title",
          ["first",
            ["hops", {
              "datasets": ["oslo-bysikkel-stations t1"],
              "where": [
                ["eq", "_S.id", "t1.id"]
              ],
              "return": "t1.title"
            }]
          ]
        ]
      ]
    }
  }],
  "pump": {
    "schedule_interval": 900
  }
}
```
Press "Save" and "Start" buttons.

Now station availability data will be fetched and stored in Sesam every 15 minutes. 

That's all, right now bicycle season is closed due to winter, but will be opened from 1 March. We will collect some data
enough for availability analysis, but this will be topic for next post.