---
layout: post
title: Writing Groovy Script Libraries in SoapUI 
categories: Testing
tags: [Groovy, testing, SoapUI]
author: dalibor
---

# Motivation and target audience

Very often when we are creating tests in [SoapUI](https://www.soapui.org/) where we end up writing a lot of [Groovy](http://groovy-lang.org/) code to add extra flexibility to our tests, create central or shared assertions, load files and data from disk or simply add extra level of reusability to our tests. So this article is specially targeted towards API testers working with [SoapUI](https://www.soapui.org/) tool by [SmartBear](https://smartbear.com/) enabling them to reuse their [Groovy](http://groovy-lang.org/) code in various projects without resolving to, developers hated, copy-paste technique.

# Introduction

Although [SoapUI](https://www.soapui.org/) is a nice API testing tool, that has a quite extensive [Groovy](http://groovy-lang.org/) scripting support, it is not essentially designed to be used as a platform that facilities code development. Rather as a declarative environment that enables test engineers to declare [Test Cases](https://www.soapui.org/docs/functional-testing/structuring-and-running-tests.html#1-Test-Structure) and assertions quickly. However resolving to scripting capabilities is very often inevitable, when we have to load test data from database, CSV files, etc. without resolving to the paid version of [SoapUI](https://www.soapui.org/). In a multi project environment, in such cases, very often engineers had to resolve to copy+past technique when they wan to apply the same [Groovy](http://groovy-lang.org/) code on different [SoapUI](https://www.soapui.org/) projects. One way to work around this problem is to write the shared code into form of [JAR](https://en.wikipedia.org/wiki/JAR_(file_format)) library that can then be imported in different project. However this can be very impractical and demanding if our code is referencing specific [SoapUI](https://www.soapui.org/) features or session information. So here we will see how to create shared code in a more [SoapUI](https://www.soapui.org/) native way.

# Developing shared Groovy Libraries

Process of developing shared [SoapUI](https://www.soapui.org/) libraries can be divided into following steps:

1. Creating shared project with common functionality,
2. Creating [Groovy](http://groovy-lang.org/) classes with library methods,
3. Defining library loading code in project that is going to use shared library,
4. Using shared library code in specific [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html).

## Shared Project

First step in developing shared [Groovy](http://groovy-lang.org/) script library in [SoapUI](https://www.soapui.org/) is by creating an ordinary [SoapUI](https://www.soapui.org/) project that will contain one or more [Test Suites](https://www.soapui.org/docs/functional-testing/structuring-and-running-tests.html#1-Test-Structure) that will contain one or more [Test Cases](https://www.soapui.org/docs/functional-testing/structuring-and-running-tests.html#1-Test-Structure) composed of [Groovy Test Steps](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html).

![Example creating shared project](/images/2018-11-23-scriptlibrary-in-soapui/Shared_project.png) Fig. 1\. Example creating shared project

## Library Groovy classes

When we have defined [Groovy Test Steps](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) within the [SoapUI](https://www.soapui.org/) project that we wanted to share, we have to define a [Groovy](http://groovy-lang.org/) class that will contain methods that we want to reuse. We define one class for each [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) that we want to share. Class has to have a constructor that accepts: **log**, **context** and **testRunner** parameters. This parameters are actually global objects defined by [SoapUI](https://www.soapui.org/) and made available to each [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html).

En example of such a constructor can be seen here:

```groovy
/**
 * Common utility functions library
 */
class CommonUtils {

  //Global objects
  def log
  def context
  def testRunner

  def CommonUtils(log, context, testRunner) {
      this.log = log
      this.context = context
      this.testRunner = testRunner
  }
}
```

This constructor gets [SoapUI](https://www.soapui.org/) global objects when initialised and used within another [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html).

Now we can define one or more reusable methods and finally we have to define initialisation block at [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) end. Like in this example:

```groovy
CommonUtils initObj = context.getProperty("CommonUtils")
if (initObj == null) {
    initObj = new CommonUtils(log, context, context.getTestRunner())
    context.setProperty(initObj.getClass().getName(), initObj)
}
```

This block of code first checks if the object has already been initialised to assure singleton within calling context, and if the object hasn't been initialised than object is created and registered as a property of the global [SoapUI](https://www.soapui.org/) **context** object. [SoapUI](https://www.soapui.org/) generates **log**, **context** and **testRunner** objects for each test run.

So complete [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) that contains reusable [Groovy](http://groovy-lang.org/) class with public methods can look like this:

```groovy
import java.util.zip.GZIPInputStream

/**
 * Common utility functions library
 */
class CommonUtils {

  //Global objects
  def log
  def context
  def testRunner

  def CommonUtils(log, context, testRunner) {
      this.log = log
      this.context = context
      this.testRunner = testRunner
  }

  /**
   * Function that is decompressing compressed XML messages
   */
  String unzip(byte[] raw, String encoding = "UTF-8") {
    if (raw.length == 0) return ""

     String asString = new String(raw)

    def idx = asString.indexOf("<soap11:Envelope")
    if (idx >= 0) {
        return asString.substring(idx);//not zipped
    }
    idx = asString.indexOf("<Envelope")
    if (idx >= 0) {
        return asString.substring(idx);//not zipped
    }
    idx = asString.indexOf("<soapenv:Envelope")
    if (idx >= 0) {
        return asString.substring(idx);//not zipped
    }

    for (int i = 0; i < raw.length; i++) {        
        if (raw[i] == 31) {
            def length = raw.size()-1
            def messagePart = raw[i..length]

            def inflaterStream = new GZIPInputStream(new ByteArrayInputStream(messagePart.toArray(new byte[messagePart.size()])))
                def uncompressedStr = inflaterStream.getText(encoding)

            log.info "DEBUG: unzip message uncompressedStr=$uncompressedStr"

            return uncompressedStr;
        }
    }

    return "Error unzipping message"
  }
}

/* Initialisation block */
CommonUtils initObj = context.getProperty("CommonUtils")
if (initObj == null) {
    initObj = new CommonUtils(log, context, context.getTestRunner())
    context.setProperty(initObj.getClass().getName(), initObj)
}
```

## Library loading code

Now that we have defined the class and initialisation block, we need a code that will run that block of code from another [SoapUI](https://www.soapui.org/) project, and thus load classes that current project will use. Current project shared library object loading block is best placed in its own disabled [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) that can be placed in specific **Initialisation** Test Case. This Test Case can contain initialisation code for both local shared library code and global one.

Example of this organisation structure for the [SoapUI](https://www.soapui.org/) project can be seen on following screen shot:

![Structure of the SoapUI project that is using shared library](/images/2018-11-23-scriptlibrary-in-soapui/InitLib.png) Fig. 2\. Structure of the [SoapUI](https://www.soapui.org/) project that is using shared library

The code in the InitLib [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) is first getting right [SoapUI Workspace](https://www.soapui.org/getting-started/working-with-soapui/managing-workspaces.html) by using globally available **testRunner** object. From that **workspace** object, script is trying to get shared library **project** object. The code works differently when the script is invoked from within [SoapUI](https://www.soapui.org/) GUI environment, or when the script is invoked from the command line, or when running on [Jenkins](https://jenkins.io/) server without GUI.

After assuring that shared library project has been correctly opened (here again process is different when working within GUI or from command line), the script is executing shared library initialisation block (see [above](#library-groovy-classes)) and thus effectively creating shared library class and storing reference to it as a property of the **context** object.

The whole library loading code is give here:

```groovy
import com.eviware.soapui.impl.wsdl.WsdlProject
import java.util.HashMap
import java.util.Map

/* LIBRARY INTIALIZATION BLOCK */
def project = null
def workspace = testRunner.testCase.testSuite.project.getWorkspace();

//Defining initialization steps to run
def scriptLibNames = ["ErrorHandling", "CommonUtils"]
Map<String, Object> scriptLibrary = new HashMap<>()

//if running Soapui
if(workspace != null){
  project = workspace.getProjectByName("ScriptLibrary")
}
//if running in Jenkins
else{
  project = new WsdlProject("src/test/soapui/EMIF-library.xml");
}

if(!project.open) {
    project.reload()

    //if running Soapui
    if(workspace != null){
          project = workspace.getProjectByName("ScriptLibrary")
    }
    //if running in Jenkins
    else{
          project = new WsdlProject("src/test/soapui/EMIF-library.xml");
    }
}

//make a connection to the ScriptLibrary project
if(project.open && project.name == "ScriptLibrary" ) {

  def lib = project.getTestSuiteByName("library").getTestCaseByName("common")

  if(lib == null) {
    throw new RuntimeException("Could not locate ReusableScripts! ");
  }
  else{

      scriptLibNames.each() { stepName ->
      testStep = lib.getTestStepByName(stepName)      
      testStep.run(testRunner, context)      
       scriptLibrary.put(stepName, context.getProperty(stepName))
       //log.warn "Putting ${stepName} with object" + context.getProperty(stepName)
      }
  }   
}
else{
  throw new RuntimeException("Could not find project 'ScriptLibrary' !")
}

/* Storing reference to map with all loaded obects as a property of Context object */
scriptLibraryChk = context.getProperty("scriptLibrary")
//log.info scriptLibraryChk
if (scriptLibraryChk == null) {
    context.setProperty("scriptLibrary", scriptLibrary)
    //log.info scriptLibrary
}
```

## Using shared library

Finally in our project specific [Groovy Test Step](https://support.smartbear.com/readyapi/docs/soapui/steps/groovy.html) we can start using our shared library by invoking following code:

```groovy
if (context.getProperty("scriptLibrary") == null)
        testRunner.testCase.testSuite.getTestCaseByName("Initialization").getTestStepByName("InitLib").run(testRunner, context)    

def commonUtils = context.getProperty("scriptLibrary").get("CommonUtils")

def originalResponse = commonUtils.unzip(testRunner.testCase.getTestStepByName(stepName).getTestRequest().messageExchange.rawResponseData)
```

This code first checks if the library has been loaded already (to assure singleton within the calling context) and than it loads the objects by running shared library loading script that initialises all shared library objects and stores them in a map object that is available as a property of the **context** object.

# Conclusion

Unfortunately [SoapUI](https://www.soapui.org/) [Groovy](http://groovy-lang.org/) support does not allows for easy [Groovy](http://groovy-lang.org/) code editing, debugging and not to say [Groovy](http://groovy-lang.org/) library creation. Thus we have to stick to workarounds that are using [Groovy](http://groovy-lang.org/) code to invoke other scripts that are residing in different projects within the same workspace. This workaround is not easy and straitforward to implement, but nevertheless it allows us to have better [Groovy](http://groovy-lang.org/) script code reusability and thus less error prone code, with minimal duplication.
