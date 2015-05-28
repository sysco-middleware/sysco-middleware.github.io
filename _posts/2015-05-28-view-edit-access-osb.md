---
layout: post
title: Logged in role is not allowed to view/edit Access Control Policies in OSB
categories: OSB Access Control Policies
tags: [console, osb, security]
author: jphjulstad
---
If you want to change Access Control Policies in OSB on 12.1.3 it works fine on the Quickstart, but in the regular install you will probably get into the same problem as me. Fortunately I fould the solution on MOS: Cannot open the Policy Editor due to, "Logged in role is not allowed to view/edit Access Control Policies." (Doc ID 1963087.1)

A user which is a part of the Administrators group is unable to open the policy editor with the hint, "Logged in role is not allowed to view/edit Access Control Policies." as below.

![](/images/2015-05-28-view-edit-access-osb/console_security.png)

The reason is because the Administrators group is a member of the application role MiddlwareAdministrator. However, MiddlewareAdministrator does not have a permission AdminOnlyTaskAccess.

So what you have to do is the folowing:

1. Login to Enterprise Manager - Fusion Middleware Control 12c.
2. Select as below.
```
   WebLogic Domain
   > <domain-name> (right-click)
     > Security (Menu)
       > Application Policies (Menu)
```
3. Set as below, then click the arrow (Search application security grants) next to Principal Name.
```
   Application Stripe: Service_Bus_Console
   Principal Type: Application Role
```
4. Select MiddlewareAdministrator, then click Edit.
5. Add AdminOnlyTaskAccess to Permissions as below, then click OK.
```
   Permission Class: oracle.soa.osb.console.common.permissions.OSBPermission
   Resource Name: AdminOnlyTaskAccess
   Permission Actions: update
```   