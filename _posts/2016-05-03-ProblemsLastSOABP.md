---
layout: post
title: Problems with latest SOA 12.1.3 BP (160419)?
categories: oracle soa
tags: [Oracle, Patching, SOA]
author: jphjulstad
---

# Wrong patch uploaded
Did you have problems with the latest SOA 12.1.3 Bundle Patch 22970958 (12.1.3.0.160419). We also. When you download the patch - look at inventory.xml and see that it contains the following information:

```xml
<date_of_patch year="2016" month="Apr" day="6" time="08:31:40 hrs" zone="PST8PDT"/> 
```

and not this:

```xml
<date_of_patch year="2016" month="Mar" day="24" time="18:05:47 hrs" zone="PST8PDT"/> 
```

If not - ask Support to give you the correct version.