---
layout: post
title: SCA view in Migrated OSB Projects not showing correctly
categories: soa
tags: [Oracle, Oracle Service Bus, JDeveloper, Bug, Migration]
author: jphjulstad
---

# SCA view in Migrated OSB Projects not showing correctly

After migration OSB projects from versions 11 to 12.1.3 or to 12.2.1 - I have seen some of the Project views (same as the Composite view in SOA) having problems to work. Here is what it looked like:

![Empty View](/images/2016-02-19-JDev/jdev-view.png)


I think me and Christopher fould the solution today. In the JPR-file, there is a part:

```bash
  <hash n="oracle.ide.model.TechnologyScopeConfiguration">
      <list n="technologyScope">
         <string v="Maven"/>
		 <string v="SOA"/>
         <string v="ServiceBusTechnology"/>
         <string v="WSDL"/>
         <string v="WSPolicy"/>
         <string v="XML"/>
         <string v="XML_SCHEMA"/>
      </list>
   </hash>
```

If we created a new Project - we saw that the SOA-technology scope was not there. So we deleted that line, and restarted JDeveloper - and that made it work. Here is the fixed version:

```bash
  <hash n="oracle.ide.model.TechnologyScopeConfiguration">
      <list n="technologyScope">
         <string v="Maven"/>
         <string v="ServiceBusTechnology"/>
         <string v="WSDL"/>
         <string v="WSPolicy"/>
         <string v="XML"/>
         <string v="XML_SCHEMA"/>
      </list>
   </hash>
```

