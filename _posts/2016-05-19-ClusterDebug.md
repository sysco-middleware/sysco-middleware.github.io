---
layout: post
title: Cluster debugging?
categories: oracle soa
tags: [Oracle, Clustering, SOA]
author: jphjulstad
---

# Some tips when debugging cluster stability
In one environment we had problems with cluster stability - and there were unnecessary migrations of JMS Servers.

Here are some settings that gave more info in logs. Set the following for the JVM:

```
-Dweblogic.debug.DebugServerMigration=true 
 -Dweblogic.debug.DebugSingletonServices=true 
```
 
Change the below setting to DEBUG 
```
 Home >Summary of Servers >AdminServer >Logging >General >Advanced >Minimum severity to log: 
```

 enable below debugs from the weblogic admin console. 

 ```
 Debug >weblogic >core 
 cluster Collapse Node 
 DebugAsyncQueue 
 DebugCluster 
 DebugClusterAnnouncements 
 DebugClusterFragments 
 DebugClusterHeartbeats 
 DebugLeaderElection 
 DebugReplication 
 DebugReplicationDetails 
```

 Change the logging severity to Debug 
 ```
 Servers-->Server_name-->Logging-->Advanced--> Standard out :Severity level:Debug 
 Log file :Severity level:Debug 
```

Also look at Cluster >> Monitoring >> Summary to see the number of sent / received / retransmitted. For a healthy system sent and received should be almost same - and retransmissions low.