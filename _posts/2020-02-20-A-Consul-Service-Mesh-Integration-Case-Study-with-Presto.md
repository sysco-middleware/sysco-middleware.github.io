---
layout: post
title: A Consul Service Mesh Integration Case Study with Presto
categories: Hashicorp, Integration, Service Mesh
tags: [Consul, Hashicorp, Nomad, Presto, Service Mesh, Connect]
author: gugalnikov
---

# Introduction

This is a follow-up post to the presentation we held yesterday at Hashitalks 2020:

[YouTube Stream](https://www.youtube.com/watch?v=xV4JnJgOHXE)

[Slideshare](https://www.slideshare.net/FranciscoArturoViver/a-consul-service-mesh-integration-case-study-with-presto)

The main point being that we're actually leveraging these technologies as part of a wider initiative to build a state-of-the-art data platform. 

# 1. Presto

Presto is a distributed SQL tool which has become a key element in our toolbox because of its flexibility and growing number of plugins.

![](/images/2020-02-20-A-Consul-Service-Mesh-Integration-Case-Study-with-Presto/presto.png)

# 2. Consul Connect

Consul Connect is the service-mesh implementation provided by Hashicorp. At its simplest, it allows you to easily secure communication between all services in your ecosystem by using sidecar proxies which handle any inbound / outbound connections, while the services themselves will only talk to their own proxy through the loopback interface.

![](/images/2020-02-20-A-Consul-Service-Mesh-Integration-Case-Study-with-Presto/connectsidecar.png)

When dealing with opinionated services or applications (meaning that they will impose an architectural burden on you), the traditional blueprint can be severely strained, for example because the coordinator will use proprietary methods to differentiate among its different workers / brokers, thus rendering the cluster unable to communicate internally through loopback. 

There's another option though called "Connect Native Integration", which we ended up exploring and adopting when we found ourselves facing a similar challenge with Presto:

![](/images/2020-02-20-A-Consul-Service-Mesh-Integration-Case-Study-with-Presto/consulconnect.png)

In many cases, the features offered by this kind of "opinionated" application, outweigh by far the architectural shortcomings; Kafka / Zookeper would be another good example of this. 

So, native integration becomes a very viable option for keeping architectural integrity within your Control Plane while still leveraging such technologies as part of your ecosystem.

# 3. Connect-Native integrating Presto

Native integration is basically mutual TLS + an API Authorization call, so we decided to contribute some code to the Presto open-source distribution [prestosql.io](prestosql.io), in order to make the Certificate handling functionality pluggable as well. 

![](/images/2020-02-20-A-Consul-Service-Mesh-Integration-Case-Study-with-Presto/contrib.png)

So, there's an open pull-request towards the Presto master (with a quite good outlook) and a ConsulConnect plugin (which does the API call) we have also developed and should be the next merge request.

The resulting architecture is something we really like, because it is a clean and organized way to solve a very challenging problem; also, it works beautifully!!

![](/images/2020-02-20-A-Consul-Service-Mesh-Integration-Case-Study-with-Presto/architecture.png)

# 4. Key Takeaways

All in all, we have learnt a lot of lessons throughout this journey, so these are our 3 main conclusions:

- State-of-the-art should be "simple as that"
- Refactor to reduce, redesign to refine
- Fortune favors the bold

All of the latter speak to the importance of simplicity and also the benefits of embracing a mindset which allows our tech initiatives to be iterated upon, improved and re-shaped while striving to deliver top quality & value.

Thanks for reading!!
