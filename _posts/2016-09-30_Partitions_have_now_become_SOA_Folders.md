---
layout: post
title: Partitions have now become SOA Folders
categories: SOA Suite
tags: [SOA]
author: catoaune
---
Have you upgraded to Oracle SOA Suite 12.2.1.1 and can't find the Partitions configuration any longer?

Between Oracle SOA Suite 12.2.1.0 and 12.2.1.1 Partitions have been renamed to SOA Folders.

![From 12.2.1 docs](images/2016-09-30_Partitions_have_now_become_SOA_Folders/index_1221.png)

![From 12.2.1.1 docs](images/2016-09-30_Partitions_have_now_become_SOA_Folders/index_12211.png)

![From 12.2.1 docs](images/2016-09-30_Partitions_have_now_become_SOA_Folders/doc_1221.png)

![From 12.2.1.1 docs](images/2016-09-30_Partitions_have_now_become_SOA_Folders/doc_12211.png)

Why would Oracle do this in a minor update?

Good question, one reason could be to prepare for multitenancy. In WebLogic multitenancy, they use the name Partitions for the multitenant "areas", so using Partitions several places meaning different things could be rather confusing.
