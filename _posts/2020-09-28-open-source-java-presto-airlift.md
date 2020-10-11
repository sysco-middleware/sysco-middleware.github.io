---
layout: post
title: Java Open Source Contributions to Presto & Airlift
categories: Hashicorp, Integration, Service Mesh, Java, Presto
tags: [Consul, Hashicorp, Java, Presto, Service Mesh, Connect, SPIFFE]
author: gugalnikov
---

# Introduction

This is a follow-up to a previous post describing the work we are doing towards constructing a state-of-the-art data platform from the ground up:

[Previous post](https://blog.sysco.no/hashicorp,/integration,/service/mesh/A-Consul-Service-Mesh-Integration-Case-Study-with-Presto)

Here I will describe a couple of key contributions we developed and merged into the [Presto](https://github.com/prestosql/presto) & [Airlift](https://github.com/airlift/airlift) codebases in order to be able to fully incorporate these technologies into our project.

# 1. Contributing to an Open Source Project

In my opinion, every time you decide to incorporate an open source component into an enterprise system, it is a very good practice to not only read through the documentation and examples thoroughly, but also look closely into the codebase where you can easily spot a few important things:

- Is the project alive and kicking?, meaning that contributions, commits, resolved issues, merged PRs, etc. are still happening at a decent frequency over the last few months
- Code quality: is the project encouraging developers to comply with standards, naming conventions, healthy practices?, does it provide any guidelines together with a reasonable code of conduct?
- Testing: Is there a CI/CD pipeline with automated testing that is always triggered when adding or modifying code (e.g. Gtihub Actions)?, does it include regression testing and other types of useful probes?, when problems and errors occur during automated testing, it this being reported in a human readable way which facilitates correction?
- Release lifecycle: Is there a sound strategy for packaging and releasing new versions?, for example having Alfa / Beta releases besides the latest stable package which can be deployed and perfected through the communitiy's feedback.
- Code owner engagement: Are some of the key developers of the project available for discussions, code review, issue tracking, etc?, is there a way to have a deeper technical conversation with them in case it becomes necessary (e.g. a Slack team)?

The more boxes you check when looking at the codebase, the more trust you can have in the project and its viability to become a vital part of your own system. Furthermore, it is important to make sure that if / when needed: small adjustments can be made, bugs can be corrected, patches can be issued, additional features which make sense can be developed, documentation can be improved and so on in an iterative & agile fashion.

That being said, you can always take the next step and actually contribute to the project. Even the smallest form of collaboration can make a big difference and benefit the whole community, for example: submitting improvements to the documentation, voting up and / or commenting on reported issues, reviewing relevant PRs, and of course writing code for new features, bugfixes, patches, etc.

Almost in any case, in order to contribute efficiently it is useful to have a good grip on the Git flow as well as on concepts such as squashing, rebasing and so on.

![](/images/2020-09-28-Presto-Airlift/GitHub-Flow.png)

In the case of Airlift & Presto, these are very well-structured Java projects (especially Presto), where most things are highly automated through bots and actions, from signing the CLI to getting your stuff thoroughly tested via pre-baked docker compose scenarios which include not only a single instance topology but also production grade configurations including critical dependencies such as Hive, HDP, etc. All of this makes it super easy to just focus on the task at hand and move along the paces until the code is ready for review and promotion from the maintainers.

The two contributions described on this blogpost were a bit challenging to get on an official release because they meddle with the core functionality of the products, so there was a lot of back and forth to make sure that whatever breaking changes we were introducing could be easily managed through configuration and didn't create major issues for current users of these technologies. 

# 2. Adding a Pluggable Certificate Authenticator to Presto

[Link to the original PR](https://github.com/prestosql/presto/pull/3804)

Our PR addressed the following concerns:

- Related to Prestosql issue [2117](https://github.com/prestosql/presto/issues/2117) to some extent
- Allows customization of the "CERTIFICATE" authentication method by registering plugins (similar to PasswordAuthenticator)
- This can be useful in many cases, when some additional security checks must be done using the certificate's data (serial number, subject alternative names, etc.) besides only obtaining the principal
- Our specific use case is service mesh native integration (e.g. Consul Connect), where once mutual TLS is established at a JKS level, the serial number & SPIFFE Id must be obtained and vaildated for authorization by means of an API call to a service agent

![](/images/2020-09-28-Presto-Airlift/mutualtls.png)

This functionality was made available from release 334 after the PR was approved and merged. 

# 3. Making SSL Hostname Verification Configurable for Airlift's Embedded Jetty

[Link to the original PR](https://github.com/airlift/airlift/pull/858)

- Embedded Jetty allows enabling / disabling SSL hostname verification through config mechanisms; however, within Airlift, the "setEndpointIdentificationAlgorithm" method is hardcoded with an "HTTPS" value, which ensures hostname verification will always be performed. 

- As a result of this, any system which uses airlift as a foundation for internal communication won't have any control over this, even if it configures its own trust manager or uses the available JVM property on startup (-Djdk.internal.httpclient.disableHostnameVerification)

- This is a very simple change adding a config property which allows airlift users to leverage embedded Jetty's configuration possibilities.

- The justification is that there are use cases which require disabling the hostname verification for SSL (or don't really need it), without bypassing the whole certificate chain trust process. 

- A good example is a system or application which subscribes to the [SPIFFE / SPIRE](https://spiffe.io/) standard, where certificate chain trust is enforced but vanilla hostname verification (based on certificate CN) is not necessary as the SPIFFE URI in the certificate's SAN is a much more effective and flexible way to prevent the man-in-the-middle attack.

This functionality is available from release 198. 

# 4. Key Takeaways

- We really learnt a lot from doing these contributions and it was very satisfying to see them finally make it into the main codebases and official releases; this is after all technology which was originally developed in the big leagues (Facebook), so it makes us really proud to be able to make an impact at such a level of innovation.
- Getting involved with the technology we're using in such a way has given us a huge boost in our own innovation journey, both because we got the functionality we needed but also because now we have a much better understanding not only of the tools but also of our own system. Furthermore, we were able to incorporate some of the nice development practices we learnt from Presto & Airlift into our own toolset.
- On a personal note, I truly believe that this is the right way to go about building modern systems in this day and age; with such a competitive landscape, so many options at our disposal, increasingly challenging requirements and especially the need for dramatically reducing time-to-market, we can hardly afford to get stuck in the old ways and impose constraints on ourselves which will inhibit collaboration and the ability to automate, iterate and improve continuosly.

Thanks for reading!!
