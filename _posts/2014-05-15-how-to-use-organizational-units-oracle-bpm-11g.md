---
layout: post
title: How to use Organizational Units on Oracle BPM 11g
categories: bpm
tags: [human, task, 11g, oracle, soa, bpm, javaee, dvm]
author: jeqo
---

In BPM projects, Organizational Units (OU) describes the departments or offices into a company that are involved in a business process. These departments or user groups maybe are not in an "official" organizational chart necessarily, but they have a meaning to the business process.

When a project is created using Oracle BPM Studio, there is a file called "organization.xml" that is created. This file contains the roles, organizational units, calendars and holidays.

![organization.xml][/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1157.png]

## Case Study: Assign a Task by Organizational Unit

In business processes normally there are cases when you need to approve one task but only the participants related with a department, e.g.: If one vacation request must be approved by the department manager. If one is from a Marketing employee, the Marketing manager should approve that request.

### Creating the Process

1. We will create a BPM project with a process as simple as the following:

![BPMN Project](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1209.png)

2. Then we can define a Business Object (with or without a XML Schema) like this one:

![Business Object](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1212.png)

3. Implement a Human Task using the wizard:

![Auto-generate UI](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1219-610x391.png)

![Project created](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1221-610x391.png)

We can run this process with "weblogic" user as a participant in the "approver" role and test it.

### Adding Users and Organizational Units

After testing the process, we can add some users and OUs.

1. To create users, we can use the administration console from WebLogic or we can use WLST. Check this post from Edwin Biemond: [http://biemond.blogspot.no/2010/01/creating-users-and-groups-in-weblogic.html](http://biemond.blogspot.no/2010/01/creating-users-and-groups-in-weblogic.html)

In this case, I have created these users:

![Users](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1229-610x422.png)

And all these ones belong to the group "approvers"

2. Double click on "Organization" into the BPM project. Create these Organizational Units:

![Organization Units](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1233-610x391.png)

3. Assign the WebLogic group to the BPM role:

![Roles](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1236-610x378.png)

We can test the process now. All the users should have access to one task until some user claims it.

But, if the requirement is "The requests must be assigned to and approved by the Department Manger" we can use different approaches: Get the manager from a Business Rule task, from a Service, call a Business rule directly from the Human Task, we can use Parametric Roles, or use Organizational Units :D.

I think that when the requirement says "by department" or "by office" we should use Organization Units. Here is how to:

### Using Organizational Units

1. Select which Organizational Unit must be related with the BPM project. When the OU is in the project, use the second option, if the OU is in BPM Workspace use the third one:

![OU Mapping](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1249-610x157.png)

We must check that all the users, groups and roles used in the assignments must be part of the Organizational Units under the selected root (including the root). If a participant is not included, the task will not appear on BPM Workspace.

2. Then, we should tell the process with which OU must work:

![OU Data Association](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1255-610x457.png)

But what if the ID attribute coming from the Database is not related with my Organizational Unit name? In these cases we can use a Domain Value Map. This artifact let us define a relation between different domains. With this is not necessary to create tables to map values, or create boiler-plate code e.g. if we have a code on SAP and another one on BPM, we can use a DVM to match them.

3. Create a DVM and add two columns "DB" and "BPM"

![DVM](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1309.png)

Once created we can add values:

![DVM Values - 1](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1312.png)

![DVM Values - 2](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1314.png)

![DVM Values - 3](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1310-610x385.png)

4. We can use XPath functions to get values from DVM artifacts. Let's edit the data mapping in the start event:

![DVM Mapping](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1335-610x457.png)

The function should have this parameters:

```xml
    dvm:lookupValue("dvm/Department.dvm", "DB", string(bpmn:getDataOutput('request')/ns:departmentId), "BPM", "ACME")
```

![DVM Function](/images/2014-05-15-how-to-use-organizational-units-oracle-bpm-11g/2014-05-15_1336-610x528.png)

Now we can deploy and test the task assignments. The task must be assigned only to the Department approver.

Source Code: [https://github.com/jeqo/bpm_org_poc](https://github.com/jeqo/bpm_org_poc)