---
layout: post
title: Setting Up and Initializing Database Connection from Groovy Test Step in SoapUI
categories: Testing
tags: [Groovy, testing, SoapUI]
author: dalibor
---

# Introduction

Although [SoapUI](https://www.soapui.org/) has a [Test Step](https://www.soapui.org/docs/functional-testing/working-with-teststeps.html) called "[JDBC Request](https://www.soapui.org/jdbc/reference/the-jdbc-request-window.html)" this test step requires hard coded credentials to run properly. Otherwise we are going to get "java.sqlSQLException" because of missing credentials. However very often we don't want to supply hard coded credentials. Especially if the tests are running on test automation server (e.g. from [Jenkins](https://jenkins.io/)). Sometimes we want to apply [Java cryptography](https://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html) facilities for added security. This can only be applied from within [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) by using [Groovy](http://groovy-lang.org/) code and appropriate [Java](https://www.java.com/) classes.

In this article we are going to demonstrate how to use [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) within [SoapUI](https://www.soapui.org/) to load database credentials from disk, and than initialize database connection by using [Groovy](http://groovy-lang.org/). At the and we will see practical example on using this database connection.

# Setting Up and Initializing Database Connection

To set up database connection from within [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) we are going to do following steps:

1. Create Initialization [Test Case](https://www.soapui.org/docs/functional-testing/structuring-and-running-tests.html#1-Test-Structure) with disabled [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html),
2. Load credentials from disk,
3. Set up and load database driver,
4. Save database connection as a property.

## Create Initialization Test Case with disabled Groovy Test Step

First we have to prepare our test environment. Since many [Test Cases](https://www.soapui.org/docs/functional-testing/structuring-and-running-tests.html#1-Test-Structure) within our project will share the same database connection it is best to store database initialization [Test Step](https://www.soapui.org/docs/functional-testing/working-with-teststeps.html) in a separate [Test Case](https://www.soapui.org/docs/functional-testing/structuring-and-running-tests.html#1-Test-Structure). This [Test Case](https://www.soapui.org/docs/functional-testing/structuring-and-running-tests.html#1-Test-Structure) can be used to store initialization code for shared libraries as well.

Example of such structure is visible on the following picture:

![Structure of the project that is using shared library and database initialization](/images/2018-11-27-db-connection-in-soapui/Project_structure.png) Fig. 1\. Structure of the [SoapUI](https://www.soapui.org/) project that is using shared library and database initialization [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html)

## Load credentials from disk

We don't want hardcoded credentials in our code or in [JDBC Request](https://www.soapui.org/jdbc/reference/the-jdbc-request-window.html) test step. That's why we are going to load them from disk. Code example to do so can be seen below:

```groovy
class Credentials {
    String userName
    String password
}


//When running in SoapUI
String fileName = "/Users/<username>/Documents/elemif/src/test/resources/database.json"

//When running on Jenkins (DISABLE when needed)
fileName = "src/test/resources/database.json"


def inputFile = new File(fileName)
Credentials credentials = new JsonSlurper().parseText(inputFile.text)
```

When we are running our tests from within [SoapUI](https://www.soapui.org/) GUI environment we have to supply absolute paths but when we are running our tests from command line or from e.g. [Jenkins](https://jenkins.io/) server than we have to use relative paths.

## Setting up and loading database driver

In order to load database driver from [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) we first have to place database driver in a correct location. When running this test step from [SoapUI](https://www.soapui.org/) GUI than we have to find installation folder of [SoapUI](https://www.soapui.org/) (On my mac this is: `/Applications/Development/SoapUI-5.4.0.app)` and put the driver in: `Contents/java/app/bin/ext`. On Windows or Linux machine the path can be a little bit different but it will still be `bin/ext` folder.

Example with updated Oracle JDBC (ojdbc8.jar) dirver location can be seen on the following picture:

![Driver location](/images/2018-11-27-db-connection-in-soapui/Driver-location.png) Fig. 2\. Driver location

When we have driver that we want to use in place we are now able to load and initialize driver from [Groovy](http://groovy-lang.org/) code with this script:

```groovy
com.eviware.soapui.support.GroovyUtils.registerJdbcDriver("oracle.jdbc.driver.OracleDriver")
Sql sql = Sql.newInstance(
    'jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=xxx.xxx.xxx.xx)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=<service_name>)))',
    credentials.userName, credentials.password, 'oracle.jdbc.driver.OracleDriver')
```

Finally in order to share the object we are going to register it as a property of **context** object, as can be seen from the whole script seen here:

```groovy
import groovy.sql.Sql
import groovy.json.JsonSlurper

/* DATABASE INTIALIZATION BLOCK */
class Credentials {
    String userName
    String password
}


//When running in SoapUI
String fileName = "/Users/<username>/Documents/elemif/src/test/resources/database.json"

//When running on Jenkins (DISABLE when needed)
fileName = "src/test/resources/database.json"


def inputFile = new File(fileName)
Credentials credentials = new JsonSlurper().parseText(inputFile.text)
//log.info "DEBUG: credentials=${credentials.userName}"

com.eviware.soapui.support.GroovyUtils.registerJdbcDriver("oracle.jdbc.driver.OracleDriver")
Sql sql = Sql.newInstance(
    'jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=xx.xx.xx.xx)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=custom_qa)))',
    credentials.userName, credentials.password, 'oracle.jdbc.driver.OracleDriver')

context.setProperty("sql", sql)
```

# Using database connection

To actually use database connection we have first to execute disabled [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) that contains previously described initialization code, and than we can pass captured reference to database connection **sql** object to another function that is going to use it. Here is en example how to do it from [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html):

```groovy
import com.eviware.soapui.impl.wsdl.teststeps.WsdlTestRequestStepResult;
import groovy.sql.Sql
import java.time.LocalDateTime
import java.util.Date

/**
 * Class to runn test steps (has to be class to runn in passed context)
 */
class TestSteps {


  //Global objects
  def log
  def context
  def sql

  def TestSteps(log, context, testRunner) {
      this.log = log
      this.context = context
      this.testRunner = testRunner

    testRunner.testCase.testSuite.getTestCaseByName("Initialization").getTestStepByName("InitDB").run(testRunner, context)
    this.sql = context.getProperty("sql")
  }

  /**
   * Test steps execution function
   */
  def executeSteps(def stepNames, def expectedResults) {

    HashMap<String, WsdlTestRequestStepResult> results = new HashMap<String, WsdlTestRequestStepResult>()

    log.info "DB ARCHIVE TESTING STARTET AT: ${LocalDateTime.now().toString()}"

    stepNames.each { stepName ->
        WsdlTestRequestStepResult result = testFunctions.runTestStep(stepName);
        results.put(stepName, result);
    }

    stepNames.each { stepName ->
        String result = validateTestStep(stepName, sql, results.get(stepName), dbTestUtils.verify, expectedResults);
        log.info result
    }

    sql.close()

      log.info "DB ARCHIVE TESTING ENDED AT: ${LocalDateTime.now().toString()}"
  }

  /**
   * Run test step
   * @param stepName - Name of the test step to execute
   * @return Result of test step execution
   */
  WsdlTestRequestStepResult runTestStep(String stepName) {
    //TODO additional preconditions here

    WsdlTestRequestStepResult response = testRunner.runTestStepByName(stepName);
    return response
  }

  /**
   * Validates test step
   * @param stepName - name of Test Step e.g. "RequestStartOfSupply"
   * @param sql - reference to groovy.sql.Sql object
   * @param response - reference to the Test Result response
   */
  def validateTestStep(String stepName, Sql sql, WsdlTestRequestStepResult response) {
    //TODO implement result validation by using results from database
  }

}
```

# Conclusion

With the little bit of tricks we can very easily write reusable [Groovy](http://groovy-lang.org/) code inside [SoapUI](https://www.soapui.org/) [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) where we are going to initialize and than reuse database connection in a form of [Groovy](http://groovy-lang.org/) object. This approach allow us to effectively store database credentials on disk in possibly encrypted form, thus avoiding hardcoding these credentials in specific [JDBC Request](https://www.soapui.org/jdbc/reference/the-jdbc-request-window.html) test step. This also adds to more flexibility and code reusability within [SoapUI](https://www.soapui.org/).
