---
layout: post
title: Patching JDeveloper 12.1.3 for OSB and SOA
categories: soa
tags: [Oracle, Oracle Service Bus, JDeveloper, SOA Suite]
author: jphjulstad
---

# Keeping your JDeveloper Quickstart environment up to date
Oracle has released new patches some weeks ago, and to keep a JDeveloper updated for SOA Suite and Oracle Service Bus there are three products you could patch:

* Oracle SOA Suite
* Oracle Service Bus
* Weblogic

When you see in My Oracle Support (MOS) - you can see which ones are the recommended patches. There are also documents in MOS which are updated regularly:

* OSB 11g and 12c: Bundle Patch Reference (Doc ID 1499170.1)
* SOA 11g and 12c: Bundle Patch Reference (Doc ID 1485949.1)
* Master Note on WebLogic Server Patch Set Updates (PSUs) (Doc ID 1470197.1) 

You could also patch OPatch and JDeveloper, but I did not find this necessary here.

In this environment Patch 19707784: SOA Bundle Patch 12.1.3.0.1 was applied before. This will be detected, and that patch will be rolled back.

The patching is simplified in the 12-version, because now there is only one OPatch-folder to care about - and in version 11 there were one per product. You can read more about it here: [Oracle Documentation link](https://docs.oracle.com/middleware/1213/core/OPATC/toc.htm#OPATC101)

I use Windows environment in this example, so I set the environment variables first (run as Administrator):
```bash
SET ORACLE_HOME=D:\Oracle\Middleware1213\Oracle_Home

SET PATH=%PATH%;%ORACLE_HOME%\OPatch
```

You can verify existing patches using:

```bash
opatch lspatches
```

# Weblogic

For Weblogic I have updated to: WLS PATCH SET UPDATE 12.1.3.0.6. The WLS patch applied has patch number: 21983457. You download the file p21983457_121300_Generic.zip to any directory on your file system - go to that directory, and apply using the command "opatch apply".

```bash
D:\download\1213patch\p21983457_121300_Generic\21983457>opatch apply
Oracle Interim Patch Installer version 13.2.0.0.0
Copyright (c) 2014, Oracle Corporation.  All rights reserved.


Oracle Home       : D:\Oracle\Middleware1213\Oracle_Home
Central Inventory : C:\Program Files\Oracle\Inventory
   from           : n/a
OPatch version    : 13.2.0.0.0
OUI version       : 13.2.0.0.0
Log file location : D:\Oracle\Middleware1213\Oracle_Home\cfgtoollogs\opatch\2198
3457_Feb_14_2016_16_46_21\apply2016-02-14_16-45-54PM_1.log


OPatch detects the Middleware Home as "D:\Oracle\Middleware1213\Oracle_Home"

feb 14, 2016 4:46:25 PM oracle.sysman.oii.oiii.OiiiInstallAreaControl initAreaCo
ntrol
INFO: Install area Control created with access level  0
Applying interim patch '21983457' to OH 'D:\Oracle\Middleware1213\Oracle_Home'
Verifying environment and performing prerequisite checks...
All checks passed.

Please shutdown Oracle instances running out of this ORACLE_HOME on the local system.
(Oracle Home = 'D:\Oracle\Middleware1213\Oracle_Home')


Is the local system ready for patching? [y|n]
y
User Responded with: Y
Backing up files...

Patching component oracle.wls.libraries.mod, 12.1.3.0.0...

Patching component oracle.wls.libraries.mod, 12.1.3.0.0...

Patching component oracle.wls.server.shared.with.core.engine, 12.1.3.0.0...

Patching component oracle.wls.server.shared.with.core.engine, 12.1.3.0.0...

Patching component oracle.wls.shared.with.cam, 12.1.3.0.0...

Patching component oracle.wls.shared.with.cam, 12.1.3.0.0...

Patching component oracle.wls.libraries, 12.1.3.0.0...

Patching component oracle.wls.libraries, 12.1.3.0.0...

Patching component oracle.wls.clients, 12.1.3.0.0...

Patching component oracle.wls.clients, 12.1.3.0.0...

Patching component oracle.wls.admin.console.en, 12.1.3.0.0...

Patching component oracle.wls.admin.console.en, 12.1.3.0.0...

Patching component oracle.webservices.wls, 12.1.3.0.0...

Patching component oracle.webservices.wls, 12.1.3.0.0...

Patching component oracle.wls.workshop.code.completion.support, 12.1.3.0.0...

Patching component oracle.wls.workshop.code.completion.support, 12.1.3.0.0...

Patching component oracle.wls.core.app.server, 12.1.3.0.0...

Patching component oracle.wls.core.app.server, 12.1.3.0.0...

Verifying the update...
Patch 21983457 successfully applied
Log file location: D:\Oracle\Middleware1213\Oracle_Home\cfgtoollogs\opatch\21983
457_Feb_14_2016_16_46_21\apply2016-02-14_16-45-54PM_1.log

OPatch succeeded.

```

# Oracle Service Bus

For Oracle Service Bus the first bundle patch came recently - 12.1.3.0.1. The OSB patch applied has patch number: 22364187.

```bash
D:\download\1213patch\p22364187_121300_Generic\22364187>opatch apply
Oracle Interim Patch Installer version 13.2.0.0.0
Copyright (c) 2014, Oracle Corporation.  All rights reserved.


Oracle Home       : D:\Oracle\Middleware1213\Oracle_Home
Central Inventory : C:\Program Files\Oracle\Inventory
   from           : n/a
OPatch version    : 13.2.0.0.0
OUI version       : 13.2.0.0.0
Log file location : D:\Oracle\Middleware1213\Oracle_Home\cfgtoollogs\opatch\2236
4187_Feb_14_2016_16_25_24\apply2016-02-14_16-24-57PM_1.log


OPatch detects the Middleware Home as "D:\Oracle\Middleware1213\Oracle_Home"

feb 14, 2016 4:25:26 PM oracle.sysman.oii.oiii.OiiiInstallAreaControl initAreaCo
ntrol
INFO: Install area Control created with access level  0
Applying interim patch '22364187' to OH 'D:\Oracle\Middleware1213\Oracle_Home'
Verifying environment and performing prerequisite checks...
All checks passed.

Please shutdown Oracle instances running out of this ORACLE_HOME on the local sy
stem.
(Oracle Home = 'D:\Oracle\Middleware1213\Oracle_Home')


Is the local system ready for patching? [y|n]
y
User Responded with: Y
Backing up files...

Patching component oracle.servicebus.plugins, 12.1.3.0.0...

Patching component oracle.servicebus.plugins, 12.1.3.0.0...

Patching component oracle.osb.server, 12.1.3.0.0...

Patching component oracle.osb.server, 12.1.3.0.0...

Patching component oracle.osb.common, 12.1.3.0.0...

Patching component oracle.osb.common, 12.1.3.0.0...

Verifying the update...
Patch 22364187 successfully applied
Log file location: D:\Oracle\Middleware1213\Oracle_Home\cfgtoollogs\opatch\22364
187_Feb_14_2016_16_25_24\apply2016-02-14_16-24-57PM_1.log

OPatch succeeded.

```

# Oracle SOA Suite

For Oracle SOA Suite there has been many updates, the latest one is - 12.1.3.0.5. In the patch process, it is identified that al older patch should be rolled back before the new one is applied. The SOA patch applied has patch number: 22524811.

```bash
D:\download\1213patch\p22524811_121300_Generic\22524811>opatch apply
Oracle Interim Patch Installer version 13.2.0.0.0
Copyright (c) 2014, Oracle Corporation.  All rights reserved.


Oracle Home       : D:\Oracle\Middleware1213\Oracle_Home
Central Inventory : C:\Program Files\Oracle\Inventory
   from           : n/a
OPatch version    : 13.2.0.0.0
OUI version       : 13.2.0.0.0
Log file location : D:\Oracle\Middleware1213\Oracle_Home\cfgtoollogs\opatch\2252
4811_Feb_14_2016_16_30_21\apply2016-02-14_16-29-53PM_1.log


OPatch detects the Middleware Home as "D:\Oracle\Middleware1213\Oracle_Home"

feb 14, 2016 4:30:22 PM oracle.sysman.oii.oiii.OiiiInstallAreaControl initAreaCo
ntrol
INFO: Install area Control created with access level  0
Applying interim patch '22524811' to OH 'D:\Oracle\Middleware1213\Oracle_Home'
Verifying environment and performing prerequisite checks...
Patch 22524811: Optional component(s) missing : [ oracle.integration.bpm, 12.1.3
.0.0 ] , [ oracle.mft.apache, 12.1.3.0.0 ] , [ oracle.bpm.addon, 12.1.3.0.0 ] ,
[ oracle.bpm.mgmt, 12.1.3.0.0 ] , [ oracle.hwf.standalone, 12.1.3.0.0 ] , [ orac
le.soa.b2b.client, 12.1.3.0.0 ] , [ oracle.mft, 12.1.3.0.0 ] , [ oracle.bpm.proc
essspaces, 12.1.3.0.0 ] , [ oracle.soa.workflow.wc, 12.1.3.0.0 ] , [ oracle.oep.
examples, 12.1.3.0.0 ] , [ oracle.bpm.plugins, 12.1.3.0.0 ]

Patch [ 22524811 ] conflict with patch(es) [  19707784 ] in the Oracle Home.

To resolve patch conflicts please contact Oracle Support Services.
If you continue, patch(es) [  19707784 ] will be rolled back and the new Patch
[ 22524811 ] will be installed.


Do you want to proceed? [y|n]
y
User Responded with: Y
OPatch will roll back the subset patches and apply the given patch.
All checks passed.

Please shutdown Oracle instances running out of this ORACLE_HOME on the local sy
stem.
(Oracle Home = 'D:\Oracle\Middleware1213\Oracle_Home')


Is the local system ready for patching? [y|n]
y
User Responded with: Y
Backing up files...
Rolling back interim patch '19707784' from OH 'D:\Oracle\Middleware1213\Oracle_H
ome'

Patching component oracle.rules, 12.1.3.0.0...

Patching component oracle.soa.mgmt, 12.1.3.0.0...

Patching component oracle.integration.bam, 12.1.3.0.0...

Patching component oracle.soa.common.adapters, 12.1.3.0.0...

Patching component oracle.soacommon.plugins, 12.1.3.0.0...

Patching component oracle.integration.soainfra, 12.1.3.0.0...
RollbackSession removing interim patch '19707784' from inventory


OPatch back to application of the patch '22524811' after auto-rollback.


Patching component oracle.oep, 12.1.3.0.0...

Patching component oracle.rules, 12.1.3.0.0...

Patching component oracle.mft.client, 12.1.3.0.0...

Patching component oracle.rcu.soainfra, 12.1.3.0.0...

Patching component oracle.oep.plugins, 12.1.3.0.0...

Patching component oracle.soa.mgmt, 12.1.3.0.0...

Patching component oracle.integration.bam, 12.1.3.0.0...

Patching component oracle.soa.common.adapters, 12.1.3.0.0...

Patching component oracle.soacommon.plugins, 12.1.3.0.0...

Patching component oracle.integration.soainfra, 12.1.3.0.0...

Verifying the update...
Patch 22524811 successfully applied
Log file location: D:\Oracle\Middleware1213\Oracle_Home\cfgtoollogs\opatch\22524
811_Feb_14_2016_16_30_21\apply2016-02-14_16-29-53PM_1.log

OPatch succeeded.
```

After successful patching you cold verify:

```bash
D:\download\1213patch\p21983457_121300_Generic\21983457>opatch lspatches
feb 14, 2016 4:56:24 PM oracle.sysman.oii.oiii.OiiiInstallAreaControl initAreaCo
ntrol
INFO: Install area Control created with access level  0
21983457;
22524811;
22364187;
```



After reading MOS Note BPM 12c Bundle Patch Reference (Doc ID 1983445.1) stating that "This patch is mutually exclusive with all SOA Bundle Patches after SOA Bundle Patch 1. For example, if you attempt to apply this patch on top of SOA Bundle Patch 2, all fixes from SOA Bundle Patch 2 will be rolled back." - I would be careful applying BPM and SOA Patches in the same environment.