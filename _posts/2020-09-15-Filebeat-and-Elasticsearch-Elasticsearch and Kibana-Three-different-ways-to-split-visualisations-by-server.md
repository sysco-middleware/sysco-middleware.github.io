---
layout: post
title: Kibana and Elasticsearch - Three different ways to split visualisations by server
categories: Data analysis
tags: [Elasticsearch,Kibana,logs]
author: AnitaLipsky 
---

# Background and setup
We are setting up using the [Elastic](https://www.elastic.co/) cloud to ingest, correlate and visualise various types of logs for an application.

Doing so enables visibility, or "observability" into the application, thus aiding development and delivery.

In addition, it is fun to get to know an application using search and visualisation tools that the Elastic stack offers.

## Terminology
* Elastic cloud: a family of Elasticsearch SaaS offerings including hosted Elasticsearch
* Elastic Stack: The many products such as Elasticsearch, Kibana, Beats that take data from any source, in any format, then search, * analyze, and visualize it in real time
* [Elasticsearch](https://www.elastic.co/elasticsearch/): the search and analytics engine at the heart of the stack - store, search and analyse data, in our case logs
* [Kibana](https://www.elastic.co/kibana): the visualisation layer and user interface for the Elastic Stack - visualise, navigate and share data, in our case logs
* [Filebeat](https://www.elastic.co/beats/filebeat): single purpose data shippers, in our case used to send logs from the application servers to Elasticsearch


## The challenge

How to visualise logs based on different servers? In other words, how to “split” visualisations by server?

By visualisations, I mean charts, graphs and so on that can be created in *Kibana > Visualise* which can then be added to *Kibana > Dashboards*.

This blog post will walk through three different solutions based on the following example.


The steps are applicable for logs from a test or production environment for one particular version of your app.

###Example: IIS logs sent by Filebeat from staging and production servers to Elasticsearch

**Pre-conditions**
1. The same type of logs, in this case IIS logs, are sent using Filebeat to Elasticsearch from two different servers: staging and production.
1. The logs for both staging and production are therefore searchable by using one index pattern.


**More details:**

1. The logs for the staging server have been ingested by *Kibana > Home > Add log data > Select IIS logs module > follow the steps*
1. This ingestion step was repeated for the production server
1. Both servers are sending to indices with a common name in the file path, eg "iis-anita-"
1. An index pattern based on this common name groups logs from both staging and production servers into the same index pattern, eg “iis-anita-*”


Tip: [Ingest from your first data source](https://www.elastic.co/videos/elasticsearch-service-ingest-from-your-first-data-source?token=i2q8mdjxqc)

##Three different solutions

This blog post will walk through some “proof of concept” visualisations (charts) for dashboards that show the staging VS production data in three different ways:


1. Split data within the chart using one index pattern that contains IIS data from both servers
2. Each chart is based upon a saved search that is filtered by server, then create charts
3. Create charts based on an index pattern containing data from both servers, add them to a dashboard, then filter by server on the dashboard


[TODO ADD REST OF CONTENT]


## Other solutions
These are not the only solutions, as for example it is possible to

1. create a visualisation of type "Control" which creates a drop down populated by what you wish to filter.  This Control can be added to a Dashbooard and used as a filter
2. filter data directly from each server into separate indices

Neither of these will be covered in this blog post.

## Conclusion

By enabling visibility into the application and understanding various ways of doing this enables the team to make better decisions while developing and delivering the application.

It is also fun and can really bring a team together from both sides: development and business facing.

Credit goes to the entire team for the initiative to use the Elastic stack and for the ideas and setup in this blog post.