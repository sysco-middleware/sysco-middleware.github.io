---
layout: post
title: Wrong timezone causes TopLink warning in SOA suite
categories: weblogic
tags: [weblogic, jdbc, timezone]
author: catoaune
---

From time to time we discovered SOA installations that got this little TopLink warning in the SOA server log every minute:

```bash
[TopLink Warning]: 2012.01.18 15:35:24.946--UnitOfWork(353329123)--java.lang.NullPointerException
    (String.java:147)
    at oracle.toplink.internal.platform.database.oracle.TIMESTAMPHelper.extractTimeZone(TIMESTAMPHelper.java:140)
    at oracle.toplink.platform.database.oracle.Oracle9Platform.getTIMESTAMPTZFromResultSet(Oracle9Platform.java:173)
    at oracle.toplink.platform.database.oracle.Oracle9Platform.getObjectFromResultSet(Oracle9Platform.java:145)
    at oracle.toplink.internal.databaseaccess.DatabaseAccessor.getObject(DatabaseAccessor.java:1053)
    at oracle.toplink.internal.databaseaccess.DatabaseAccessor.fetchRow(DatabaseAccessor.java:850)
    at oracle.toplink.internal.databaseaccess.DatabaseAccessor.basicExecuteCall(DatabaseAccessor.java:567)
    at oracle.toplink.internal.databaseaccess.DatabaseAccessor.executeCall(DatabaseAccessor.java:468)
    at oracle.toplink.threetier.ServerSession.executeCall(ServerSession.java:447)
    at oracle.toplink.internal.queryframework.DatasourceCallQueryMechanism.executeCall(DatasourceCallQueryMechanism.java:193)
    at oracle.toplink.internal.queryframework.DatasourceCallQueryMechanism.executeCall(DatasourceCallQueryMechanism.java:179)
    at oracle.toplink.internal.queryframework.DatasourceCallQueryMechanism.selectOneRow(DatasourceCallQueryMechanism.java:603)
    at oracle.toplink.internal.queryframework.ExpressionQueryMechanism.selectOneRowFromTable(ExpressionQueryMechanism.java:2543)
    at oracle.toplink.internal.queryframework.ExpressionQueryMechanism.selectOneRow(ExpressionQueryMechanism.java:2513)
    at oracle.toplink.queryframework.ReadObjectQuery.executeObjectLevelReadQuery(ReadObjectQuery.java:424)
    at oracle.toplink.queryframework.ObjectLevelReadQuery.executeDatabaseQuery(ObjectLevelReadQuery.java:874)
    at oracle.toplink.queryframework.DatabaseQuery.execute(DatabaseQuery.java:679)
    at oracle.toplink.queryframework.ObjectLevelReadQuery.execute(ObjectLevelReadQuery.java:835)
    at oracle.toplink.queryframework.ReadObjectQuery.execute(ReadObjectQuery.java:397)
    at oracle.toplink.queryframework.ObjectLevelReadQuery.executeInUnitOfWork(ObjectLevelReadQuery.java:899)
    at oracle.toplink.internal.sessions.UnitOfWorkImpl.internalExecuteQuery(UnitOfWorkImpl.java:2807)
    at oracle.toplink.internal.sessions.AbstractSession.executeQuery(AbstractSession.java:1079)
    at oracle.toplink.internal.sessions.AbstractSession.executeQuery(AbstractSession.java:1063)
    at oracle.toplink.internal.sessions.AbstractSession.executeQuery(AbstractSession.java:1022)
    at oracle.tip.mediator.dispatch.db.ContainerIdDBAccess.renewContainerIdLease(ContainerIdDBAccess.java:99)
    at oracle.tip.mediator.dispatch.db.DBContainerIdManager.run(DBContainerIdManager.java:188)
    at oracle.integration.platform.blocks.executor.WorkManagerExecutor$1.run(WorkManagerExecutor.java:120)
    at weblogic.work.j2ee.J2EEWorkManager$WorkWithListener.run(J2EEWorkManager.java:183)
    at weblogic.work.DaemonWorkThread.run(DaemonWorkThread.java:30)
```
	
It happened on some installations, but not on other, and we couldn’t really find any patterns that explained why it did happen on installation A and not on installation B.

After some investigation and with help from Oracle Support, we found that it was a combination of a JDBC bug and that sometimes strange things just happens.

* In the table mediator_containerid_lease, there is a timestamp with timezone column.
* For some strange reason, the JVM believed that the timezone was Atlantic/Jan_Mayen, and used the code for Atlantic/Jan_Mayen in the timestamp with timezone column.
* A bug in the JDBC driver (or some inconsistency between driver versions and DB versions) made method extractTimeZone in TIMESTAMPHelper class get a NULL result instead of the correct timezone.

Solution:

* Stop the SOA server
* Remove any rows with timezone Atlantic/Jan_Mayen from mediator_containerid_lease (we did this before the systems went into production, so we didn’t loose any valuable data)
* Start the SOA server again, but make sure to use the JVM argument * -Duser.timezone=Europe/Oslo (or some other location, but not Atlantic/Jan_Mayen)

The JDBC driver bug is fixed, so the problem should not appear on new installations anymore, but unfortunately many are not able to upgrade to the latest and greatest, so this issue might still pop up for a while.

Reference: Bug 13627118 : ATLANTIC/JAN-MAYEN MAPPING IN ZONEIDMAP.JAVA DOES NOT MATCH THE DATABASE VALUE