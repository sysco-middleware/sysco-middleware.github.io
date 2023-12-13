---
layout: post
title: Using non XA MS SQL Server driver
categories: MS SQL Server Non-XA
tags: [weblogic, jdbc, mssql]
author: jphjulstad
keep: yes
---
If you want to use the non-XA MS Sql Driver (weblogic.jdbc.sqlserver.SQLServerDriver), you must be sure that it actually does not use XA. If not, you will get error message. The way to verify this is to go into the System MBean Browser, find your JDBCSystemResource. From EM console, you go to your domain. Then right click and select System MBean Browser.

![](/images/2015-05-28-mssql-non-xa/mbean_browser.png)

And then look into the JDBC Driver Properties. To find it you must click:

```bash
JDBCResource (and value)
JDBCDriverParams (and value)
```

If UseXaDatasourceInterface is not false, you should set it to false.

![](/images/2015-05-28-mssql-non-xa/jdbcdriver_params.png)

You must then Lock & Edit,and Activate changes. WLS will then write to the appropriate JDBC configuration file.

