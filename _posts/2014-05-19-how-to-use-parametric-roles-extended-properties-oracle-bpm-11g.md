---
layout: post
title: How to use Parametric Roles and Extended User Properties with Oracle BPM 11g
categories: javaee
tags: [human, task, 11g, oracle, soa, bpm, javaee]
author: jeqo
---

In this entry, I would like to show how to use Parametric Roles, why we need Extended User Properties, some limitations and how to solve them.

Parametric Roles are user to customize one role depending on some conditions. e.g.: I want to assign one task depending on user's specializations, if one task requires check a quote related with heavy machines, this should be assigned to the supervisor specialized on heavy machines, if the quote is about light machines, should be assigned to the supervisor of light machines, and so on.

Parametric Roles use the users information founded in LDAP. This specific information could be stored on a LDAP system, but normally LDAP are only used to store basic information like e-mail accounts, names and password.

To extend the user properties, Oracle BPM offer a feature called Extended User Properties. This let us define information like location, specializations, etc.

Using this properties we can define Parametric Roles and avoid creating a "USER" table or Business Rules to define the user or users to assign a task.  

## Extended User Properties

### Creating Extended User Properties

To create and assign user properties we can use Oracle BPM Workspace application:

![Extended User Properties](/images/2014-05-19-how-to-use-parametric-roles-extended-properties-oracle-bpm-11g/2014-05-18_2333-610x315.png)

But this feature has one limitation (or bug): [Oracle Documentation](http://docs.oracle.com/cd/E14571_01/doc.1111/e15175/bpmug_ws_admin.htm#BPMUG232) says you can define four types of properties: STRING, NUMBER, DATE, and FREEFORM. And from Workspace you only can define 2: STRING and NUMBER. What if I want to define a property to store some description or address and this cannot be defined as an option list?

Well, after some tests, I have implemented a JDeveloper project to access User Properties and create NUMBER, DATE or FREEFORM properties:

```java
    public void createUserExtendedProperty(String propertyName,
                                           PropertyType propertyType) {
        try {
            beginConnection();

            IBPMServiceClient bpmServiceClient =
                clientFactory.getBPMServiceClient();

            IBPMOrganizationService bpmOrganizationService =
                bpmServiceClient.getBPMOrganizationService();

            ParticipantProperty participantProperty =
                new ParticipantProperty();

            participantProperty.setName(propertyName);
            participantProperty.setPropertyType(propertyType.name());

            bpmOrganizationService.createParticipantProperty(bpmContext,
                                                             participantProperty);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            closeConnection();
        }
    }
```

Also, is implemented how to get user assigned properties from BPM Workspace:

```java
    public List getParticipantsProperties(List participantsNames) {

        List participantsProperties =
            new ArrayList();

        try {
            beginConnection();

            IBPMServiceClient bpmServiceClient =
                clientFactory.getBPMServiceClient();

            IBPMOrganizationService bpmOrganizationService =
                bpmServiceClient.getBPMOrganizationService();

            List participantes = new ArrayList();

            for (String participantName : participantsNames) {
                PrincipleRefType principleRef = new MemberType();
                principleRef.setName(participantName);
                principleRef.setRealm("jazn.com");
                principleRef.setType(ParticipantTypeEnum.USER);

                Participant participante = new Participant(principleRef);

                participantes.add(participante);
            }

            participantsProperties =
                    bpmOrganizationService.getPropertiesOfParticipants(bpmContext,
                                                                       participantes);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            closeConnection();
        }

        return participantsProperties;
    }
```

This is the link to the Git repository:

GitHub: [JDeveloper Project](https://github.com/jeqo/bpm_org_poc/tree/master/part2/AcmeBpmApp/BPMClient)

Check the BPMClient library on the project, should have these links:

* [MW_HOME]\soa\modules\oracle.bpm.client_11.1.1\oracle.bpm.bpm-services.client.jar
* [MW_HOME]\soa\modules\oracle.bpm.client_11.1.1\oracle.bpm.bpm-services.interface.jar
* [MW_HOME]\soa\modules\oracle.bpm.client_11.1.1\oracle.bpm.client.jar
* [MW_HOME]\soa\modules\oracle.bpm.client_11.1.1\oracle.bpm.web-resources.jar
* [MW_HOME]\soa\modules\oracle.soa.workflow_11.1.1\bpm-services.jar
* [MW_HOME]\soa\modules\oracle.soa.workflow_11.1.1\bpm-workflow-datacontrol.jar
* [MW_HOME]\soa\modules\oracle.soa.workflow_11.1.1\oracle.soa.workflow.jar
* [MW_HOME]\soa\modules\oracle.soa.workflow_11.1.1\oracle.soa.workflow.wc.jar
* [MW_HOME]\soa\modules\oracle.soa.worklist_11.1.1\adflibTaskListTaskFlow.jar
* [MW_HOME]\soa\modules\oracle.soa.worklist_11.1.1\adflibWorklistComponents.jar
* [MW_HOME]\soa\modules\oracle.soa.worklist_11.1.1\adflibWorkspaceFramework.jar
* [MW_HOME]\soa\modules\oracle.soa.worklist_11.1.1\oracle.soa.worklist.jar
* [MW_HOME]\soa\modules\oracle.bpm.runtime_11.1.1\oracle.bpm.metadata.jar

## Parametric Roles

This artifacts are ideals when you have custom conditions to assign a task related with extended user properties (like specializations). In the previous example about heavy and light machines, the parametric role would have this definition: Group: Supervisor, and Extended Property: Machine Specialization.

Here you can see how to create a parametric role and test it:

1. Create a new parametric role from Oracle BPM Workspace:

![Parametric Roles](/images/2014-05-19-how-to-use-parametric-roles-extended-properties-oracle-bpm-11g/2014-05-19_1300-610x263.png)

In this case we are using some group called "approvers" and a random property called "ABC".

2. Then, we must define what Parametric Role use in the Human Task definition:

![Human Task Definition](/images/2014-05-19-how-to-use-parametric-roles-extended-properties-oracle-bpm-11g/2014-05-19_1303-610x338.png)

![Participant Type](/images/2014-05-19-how-to-use-parametric-roles-extended-properties-oracle-bpm-11g/2014-05-19_1303_001-610x583.png)

3. Then we can deploy and test the process from Enterprise Manager:

![Parametric Role - 1](/images/2014-05-19-how-to-use-parametric-roles-extended-properties-oracle-bpm-11g/2014-05-19_1307-610x401.png)

![Parametric Role - 2](/images/2014-05-19-how-to-use-parametric-roles-extended-properties-oracle-bpm-11g/2014-05-19_1308-610x119.png)

![Parametric Role - 3](/images/2014-05-19-how-to-use-parametric-roles-extended-properties-oracle-bpm-11g/2014-05-19_1308_001.png)

![Parametric Role - 4](/images/2014-05-19-how-to-use-parametric-roles-extended-properties-oracle-bpm-11g/2014-05-19_1309.png)

Source code: [https://github.com/jeqo/bpm_org_poc/tree/master/part2](https://github.com/jeqo/bpm_org_poc/tree/master/part2)