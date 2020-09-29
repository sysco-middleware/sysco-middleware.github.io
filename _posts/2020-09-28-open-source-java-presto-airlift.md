---
layout: post
title: Java Open Source Contributions to Presto & Airlift
categories: Hashicorp, Integration, Service Mesh, Java, Presto
tags: [Consul, Hashicorp, Java, Presto, Service Mesh, Connect, SPIFFE]
author: gugalnikov
---

# Introduction

This is a follow-up to a previous post describing the work we are doing towards constructing a state-of-the-art data platform from the ground up:

[Previous post](https://www.youtube.com/watch?v=xV4JnJgOHXE)

Here I will describe a couple of key contributions we developed and merged into the Presto & Airlift codebases in order to be able to fully incorporate these technologies into our project.

# 1. Contributing to an Open Source Project

In my opinion, every time you decide to incorporate an open source component into an enterprise system, it is a very good practice to not only read through the documentation and examples thoroughly, but also look closely into the codebase where you can easily spot a few important things:

- Is the project alive and kicking?, meaning that contributions, commits, resolved issues, merged PRs, etc. are still happening at a decent frequency over the last few months
- Code quality: is the project encouraging developers to comply with standards, naming conventions, healthy practices?, does it provide any guidelines together with a reasonable code of conduct?
- Testing: Is there a CI/CD pipeline with automated testing that is always triggered when adding or modifying code (e.g. Gtihub Actions)?, does it include regression testing and other types of useful probes?, when problems and errors occur during automated testing, it this being reported in a human readable way which facilitates correction?
- Release lifecycle: Is there a sound strategy for packaging and releasing new versions?, for example having Alfa / Beta releases besides the latest stable package which can be deployed and perfected through the communitiy's feedback.
- Code owner engagement: Are some of the key developers of the project available for discussions, code review, issue tracking, etc?, is there a way to have a deeper technical conversation with them in case it becomes necessary (e.g. a Slack team)?

The more boxes you check when looking at the codebase, the more trust you can have in the project and its viability to become a vital part of your own system. Furthermore, it is important to make sure that if / when needed: small adjustments can be made, bugs can be corrected, patches can be issued, additional features which make sense can be developed, documentation can be improved and so on in an iterative & agile fashion.


# 2. Adding a Pluggable Certificate Authenticator to Presto

Presto is a distributed SQL tool which has become a key element in our toolbox because of its flexibility and growing number of plugins.

![](/images/2020-02-20-A-Consul-Service-Mesh-Integration-Case-Study-with-Presto/presto.png)

# 3. Making SSL Hostname Verification Configurable for Airlift's Embedded Jetty

Consul Connect is the service-mesh implementation provided by Hashicorp. At its simplest, it allows you to easily secure communication between all services in your ecosystem by using sidecar proxies which handle any inbound / outbound connections, while the services themselves will only talk to their own proxy through the loopback interface.

![](/images/2020-02-20-A-Consul-Service-Mesh-Integration-Case-Study-with-Presto/connectsidecar.png)

When dealing with opinionated services or applications (meaning that they will impose an architectural burden on you), the traditional blueprint can be severely strained, for example because the coordinator will use proprietary methods to differentiate among its different workers / brokers, thus rendering the cluster unable to communicate internally through loopback. 

There's another option though called "Connect Native Integration", which we ended up exploring and adopting when we found ourselves facing a similar challenge with Presto:

![](/images/2020-02-20-A-Consul-Service-Mesh-Integration-Case-Study-with-Presto/consulconnect.png)

In many cases, the features offered by this kind of "opinionated" application, outweigh by far the architectural shortcomings; Kafka / Zookeper would be another good example of this. 

So, native integration becomes a very viable option for keeping architectural integrity within your Control Plane while still leveraging such technologies as part of your ecosystem.

# 3. Connect-Native integrating Presto

Native integration is basically mutual TLS + an API Authorization call, so we decided to contribute some code to the Presto open-source distribution [prestosql.io](https://prestosql.io), in order to make the Certificate handling functionality pluggable as well. 

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
