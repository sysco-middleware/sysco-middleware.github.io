---
layout: post
title: Filebeat and Elasticsearch - Adding custom fields in filebeat.yml based on log file path
categories: Data analysis
tags: [Elasticsearch,Filebeat,Kibana,ingest,logs]
author: AnitaLipsky 
---

# Overview
This will walk through adding custom fields in filebeat.yml for various log file types and paths in order to make those logs ingested by Elasticsearch more easily searchable.

## Terminology
* [Elasticsearch](https://www.elastic.co/elasticsearch/): the search and analytics engine at the heart of the stack - store, search and analyse data, in our case logs
* [Kibana](https://www.elastic.co/kibana): the visualisation layer and user interface for the Elastic Stack - visualise, navigate and share data, in our case logs
* [Filebeat](https://www.elastic.co/beats/filebeat): single purpose data shippers, in our case used to send logs from the application servers to Elasticsearch


## The log files that we will be ingesting


We will be ingesting the following log files from a staging server

Note the log file paths already contain meta information that make it easier to know information about the logs

```bash
- /var/log/*.log
- C:\ProgramData\FinancialCustomer\performance-logs\myAccountingBackOfficeApp_version_1.1\*
- C:\ProgramData\FinancialCustomer\Logs\myConnectorApp\*
- C:\ProgramData\FinancialCustomer\Logs\myTimeSheetApp_version_1.1\*
```

## The custom fields that will be added


The fields added will indicate

1. The version of the app eg version 2.1.0
2. What type of server the logs originated from eg staging or production
3. Which particular app the logs are from eg *myAccountingBackOfficeApp*, *myConnectorApp*...


Once these fields are added to the index they can be used to search and aggregate data based on these properties.

This simplifies searching for logs and creating charts aggregated by a particular version of the app on a particular server, for example if you wish to view staging logs running version 2.1.0 for *myAccountingBackOfficeApp*

## The steps to follow

For each log file type

Pre-condition: Filebeat is installed on the server containing the logs, in this case the staging server
1. Edit filebeat.yml to add the custom field for the log file
2. Verify the new fields are easily searchable in Kibana > Discover
3. Repeat the steps for the each log file type


Tip: If this is your first time... before proceeding see more detailed steps in: [Adding custom fields so ingested logs are more easily searchable](https://blog.sysco.no/data/analysis/Filebeat-and-Elasticsearch-adding-custom-fields/)

### Add custom fields for /var/log/*.log

Edit filebeat.yml to add the custom field for the log file, save, then verify the new fields are easily searchable in Kibana > Discover


```bash
# filebeat.yml --- SNIP ---
# Add the following fields and fields_under_root underneath the path

filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log
  fields:
    app.log.origin: stagingServer
  fields_under_root: true
```

The reason I used the field name app.log.origin is I am sure that there are no existing fields starting with the root name “app”.

This makes it easy for me to know which fields have been custom added, and to know that the name I chose is not overwriting an existing field and its value.


### Add custom fields for the second log file path myAccountingBackOfficeApp_version_1.1

This is done by applying [different configuration settings to different files](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-log.html#filebeat-input-log).

We need to define multiple input sections.

In filebeat.yml add the second type and its fields and fields_under_root underneath the first type, save and restart filebeat.

In this example, fields indicating the staging server, the name of the app and version will be added to every indexed document in Elasticsearch coming from the log files ```C:\ProgramData\FinancialCustomer\performance-logs\myAccountingBackOfficeApp_version_1.1\*```


```bash
# filebeat.yml --- SNIP ---
# Add the second type and its fields and fields_under_root underneath the first type

filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log
  fields:
    app.log.origin: stagingServer
  fields_under_root: true
- type: log
  enabled: true
  paths:
    - C:\ProgramData\FinancialCustomer\performance-logs\myAccountingBackOfficeApp_version_1.1\*
  fields:
    app.log.origin: stagingServer
    app.name: myAccountingBackOfficeApp
    app.version.display_name: "1.1.2020.0914"
    app.version.major: 1
    app.version.minor: 1
    app.version.patch: 2020.0914
  fields_under_root: true
```

Example: It will now be easy to filter only on *myAccountingBackOfficeApp* logs running version 1.1.0 from the staging server


### Add custom fields for the last two log file paths myConnectorApp and myTimeSheetApp_version_1.1


Repeat the steps for the last two log file types


```bash
# filebeat.yml --- SNIP ---
# Add the second type and its fields and fields_under_root underneath the first type

filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log
  fields:
    app.log.origin: stagingServer
  fields_under_root: true
- type: log
  enabled: true
  paths:
    - C:\ProgramData\FinancialCustomer\performance-logs\myAccountingBackOfficeApp_version_1.1\*
  fields:
    app.log.origin: stagingServer
    app.name: myAccountingBackOfficeApp
    app.version.display_name: "1.1.2020.0914"
    app.version.major: 1
    app.version.minor: 1
    app.version.patch: 2020.0914
  fields_under_root: true
- type: log
  enabled: true
  paths:
    - C:\ProgramData\FinancialCustomer\performance-logs\myConnectorApp\*
  fields:
    app.log.origin: stagingServer
    app.name: myConnectorApp
  fields_under_root: true
- type: log
  enabled: true
  paths:
    - C:\ProgramData\FinancialCustomer\performance-logs\myTimeSheetApp_version_1.1\*
  fields:
    app.log.origin: stagingServer
    app.name: myTimeSheetApp_version_1.1
    app.version.display_name: "1.1.2020.0914"
    app.version.major: 1
    app.version.minor: 1
    app.version.patch: 2020.0914
  fields_under_root: true
```

## Conclusion

The custom fields added to the index in Elasticsearch contain useful meta information that give the logs context.  Thus the logs can be more easily searchable in that context.

When logs are more easily searchable within a context, visual charts and graphs can easily be made for these different contexts bringing visibility into your application and enabling stakeholders to understand and act upon the application behaviour.
