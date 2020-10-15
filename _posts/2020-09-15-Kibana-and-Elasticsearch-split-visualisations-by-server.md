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

## Solution One: Split data within the chart 


1. *Kibana > Create Visualisation > Pie chart > Type > Index pattern > select the index pattern*
1. Now start creating the visualisation:  Metrics > leave the default as Slice : Count
1. *Buckets > Split Chart > Aggregation: Terms > Field: agent.hostname > Update*
1. Note: It is not necessary to update, however I do this as I go to be sure I am on the right track
1. At this point there should be two charts, one for each server. They are shown one above each other by default
1. To update the charts to be next to each other, toggle the “columns” button right under “Buckets” and Update
1. Add more details to the charts using Split Slices as follows
1. *Buckets > Add > Split Slices > Sub aggregation: Terms > Field: user_agent.os.name > Update*
1. This adds browser types to each chart
1. *Buckets > Add > Split Slices > Sub aggregation: Terms > Field: user_agent.os.version > Update*
1. This add browser versions within browser type to each chart
1. Save the chart.  It can now be added to a dashboard.

*Caption: The chart in Visualise*
![New field](images/2020-09-15-Kibana-and-Elasticsearch-split-visualisations-by-server/1 - split chart.png)

*Caption: The chart on the Dashboard*
![New field](images/2020-09-15-Kibana-and-Elasticsearch-split-visualisations-by-server/1 - split chart on dashboard.png)

**Solution One Summary: **

Splitting the chart is what actually splits the logs by server.  This chart now contains two pie charts within the same container.

Once this is put onto a dashboard, it is not possible to remove the just one of these pie charts - the whole container with both pie charts will be shown.

## Solution Two: Chart based on saved search filtered by server 


Create saved searches

1. Kibana > Discover > select the index pattern
1. Filter by the staging server by selecting the field and value for the data, eg agent.hostname : "staging-sever-123"
1. Save the search, eg "Staging server logs only"
1. Repeat these steps for the production server


Create charts

1. *Kibana > Create Visualisation > Pie chart > Type > Saved search > select one of the saved searches created earlier*
1. Now start creating the visualisation:  *Metrics > leave the default as Slice : Count*
1. This time do NOT split chart
1. Add more details to the charts using Split Slices as follows
1. *Buckets > Add > Split Slices > Sub aggregation: Terms > Field: user_agent.os.name > Update*
1. This adds browser types to each chart
1. *Buckets > Add > Split Slices > Sub aggregation: Terms > Field: user_agent.os.version > Update*
1. This add browser versions within browser type to each chart
1. Save the chart
1. Repeat the same steps for the second server
1. Add both charts to a dashboard

![New field](images/2020-09-15-Kibana-and-Elasticsearch-split-visualisations-by-server/2 - separate charts based on saved search by server.png)

*Caption: Both charts added to a dashboard*


**Solution Two Summary: **

Each chart contains only data for that one particular server.

Once this is put onto a dashboard, it is possible to remove the just one of these charts at any time.

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