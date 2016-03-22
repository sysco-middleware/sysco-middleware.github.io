---
layout: post
title: SCA view in Migrated OSB Projects not showing correctly
categories: soa
tags: [Oracle, Oracle Service Bus, JDeveloper, Bug, Migration]
author: jphjulstad
---

# SCA view in Migrated OSB Projects not showing correctly

After migrating OSB projects from versions 11 to 12.1.3 or to 12.2.1 - I have seen some of the Project views (same as the Composite view in SOA) having problems to work. Here is what it looked like:

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

Oracle has verified this as a workaround, and gave me quick feedback with details. This can also happen in new 12.2.1 project - if you add an XQuery - see details here on MOS: Adding XQuery converts Service Bus projects onto SOA projects (Doc ID 2090174.1) - with a patch.

If it does not work fine - do the following - Clear the cache of Jdeveloper with the following steps: 
 1. Open the About Oracle Jdeveloper Dialog from menu, Help -> About 
 2. In the Properties tab , search for ide.system.dir 
 3. Stop Jdeveloper and delete/rename this folder 
 4. Restart Jdev 
