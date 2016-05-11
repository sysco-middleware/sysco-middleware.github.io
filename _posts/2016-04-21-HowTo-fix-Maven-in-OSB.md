---
layout: post
title: How to fix Maven build problems in Oracle Service Bus 12c
name: HowTo-fix-Maven-in-OSB
categories: soa
tags: [Oracle, Enterprise Service Bus, Maven]
author: dalibor
---

# Introduction
To many old fashion JDeveloper developers using Maven is a new and obscure thing as JDeveloper has his own build system and does not require ANT or Maven as a build or deployment tool. However if it goes to continuous integration with many small integration automatic builds done daily than using ANT or Maven is inevitable, of which Maven is newer and more popular. Unfortunately for SOA/OSB 12c developers, other popular IDEs, like NetBeans, Eclipse or IntelliJ have much better Maven support than JDeveloper. Still relaying on his old build and deploy system, and using somewhat hybrid approach on integrating Maven, JDeveloper has a lot of issues when it comes to creating new Maven based projects and build them. Moreover Maven problem does not ends with JDeveloper when it comes to proper working of Maven with OSB architecture but it spans whole Middleware architecture.

Typical continuous integration lifecycle is composed of several phases like:
1. Merging source code files from different source control branches (typically one or more branch for each developer) into one integration branch,
2. Deploying merged and consolidated integration branch back to the source control system,
3. Executing Hudson/Jenkins build job that has been triggered by post to version control system in integration branch
4. Jenkins plugin executes Maven command line to do: compiling, testing, packaging, and deploying of different projects belonging to the same application.

Therefore we can see that for continuous integration lifecycle to work properly it is not enough to be sure that Maven works from JDeveloper but also from the command line to be able to port our build environment to dedicated integration machine.

In this article I will try to explain Maven setup and necessary workarounds in order to enable proper functionality of Maven in both JDeveloper and command line environment within our SOA/OSB 12c Middleware installation.

# Fixing JDeveloper Maven integration problems
When wi install our OSB 12c development environment JDeveloper is included in installation. When we open JDeveloper upon installation and we create our first OSB Application/Project we will see that maven build file (pom.xml) is already included in project. We can get wrong impression that Maven support is here and is working correctly. But it doesn't!

Default Maven pom.xml build file looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.oracle.servicebus</groupId>
        <artifactId>sbar-project-common</artifactId>
        <version>12.1.3-0-0</version>
    </parent>

    <groupId>MavenTest2</groupId>
    <artifactId>SBProject2</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>sbar</packaging>

    <description/>

</project>
```

For OSB developers only 'package' and 'deploy' goals of the Maven build lifecycle are relevant as OSB project does not implies actual compiling, and actual testing is done when you deploy your project on server. However when you try to run package or deploy goal by right clicking pom.xml file in Applications tree window and choosing: 'Run Maven'->package or deploy you will see that build end up in error reporting that 'com.oracle.servicebus' parent groupId is not known. Some of suggested workarounds that I have found on web and even on Oracle MySupport pages is to remove parent reference in pom.xml and change packaging from 'sbar' to 'jar'.

It is important to understand that Maven is a plugin execution framework in which every phase of execution executes different set of plugins. Some plugins are executed deliberately for one or more phases and they have to be listed in plugin section of Maven pom.xml file, but some plugins are executed implicitly. That is the case with JAR plugin invoked by specifying 'jar' in 'packaging' section of pom.xml. Default Apache JAR plugin is not suitable to build Service Bus archives as these archives have specific folder structure and content. Therefore 'sbar' has to be specified as a target in 'packaging' section as it will invoke the correct Oracle package plugin specifically designed to produce OSB archives. But how to enable it to run properly?

## Fixing Middleware pom.xml templates
Before executing any operation or rules specified pom.xml file Maven is doing several things. Maven will try to connect with one or more repositories and then download all necessary plugins needed to execute command in question. After all initial plugins have been downloaded, maven will try to merge your pom.xml file with the template forming so called effective pom.xml that is then read by Maven executor. Based on the content of effective pom.xml additional set of plugins will be downloaded and all the phases up to the phase specified on the command line will be invoked. Unfortunately for us the default set of templates appear to have buggy content and they have to be corrected. There are two ways to fix that. One is to apply latest Middleware patches and another is to do it manually by following nice blog post written by Robert Patrick that can be found here:

[https://rhpatrickblog.wordpress.com/2015/11/11/restoring-osb-12-2-1-maven-functionality/](https://rhpatrickblog.wordpress.com/2015/11/11/restoring-osb-12-2-1-maven-functionality/)

## Connecting with Oracle Maven repository
In order to enable Maven to work from within JDeveloper and command line we have to establish connection with Oracel Maven repository. That is easier said than done as due to the legal reasons Oracle Maven repository requires SSL encryption and Oracle Single Sign On to access it. Therefore we must follow these steps:

1.) Register your 'Oracle Single Sign On' account with Oracle Maven repositor:

  Open [https://maven.oracle.com](https://maven.oracle.com) web page in your browser and on that page you will have link to registration page on which you can register your SSO account with Oracle Maven repository

2.) Set ORACLE_HOME environment variable to your middleware home and not to your database home. E.g. in Linux/Unix environment:

```sh
  export ORACLE_HOME=/u01/app/Oracle/Middleware
```

  The best option is to put it in shell startup file like: .profile or .bashrc

3.) Set MAVEN_HOME environment variable to your middleware Maven home. E.g. in Linux/Unix environment:

```sh
  export MAVEN_HOME=/u01/app/Oracle/Middleware/oracle_common/modules/org.apache.maven_3.2.5
```

  Again, the best option is to put it in shell startup file like: .profile or .bashrc

4.) Download the wagon-http version 2.8 shaded JAR file from Maven Central:

  [http://central.maven.org/maven2/org/apache/maven/wagon/wagon-http/2.8/wagon-http-2.8-shaded.jar](http://central.maven.org/maven2/org/apache/maven/wagon/wagon-http/2.8/wagon-http-2.8-shaded.jar)

  This JAR file is used to enable SSL and Single Sign On functionality. We have to move that JAR file to the $MAVEN_HOME/lib/ext/ directory:

  /u01/app/Oracle/Middleware/oracle_common/modules/org.apache.maven_3.2.5/lib/ext/

5.) encrypt your Oracle Single Sign On password with:

```sh
  mvn --encrypt-master-password <password>
```

  save the results to clipboard

6.) create file: 'settings-security.xml', put it in you maven home folder (e.g. ~/.m2/) and add this element to it using encripted password saved in step 4.:

```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <settingsSecurity>
      <master>Your encripted password {jSMOWnoPFgsHVpMvz5VrIt5kR6zGpI8u+9EF1iFQyJQ=}</master>
   </settingsSecurity>
```

7.) encrypt your Oracle Single Sign On password again with:

```sh
   mvn --encrypt-password <password>
```

  save the results to clipboard

8.) Modify the content of add 'settings.xml' file as follows. This file will be generated in your Maven home folder (e.g. ~/.m2/) the first time you run any maven command.

   a.) We have to replace <server> section with the content below that includes reference to Oracle Maven repository, your 'Oracle Single Sign On' email and encrypted Single Sign On password, saved in step 6.

```xml
  <servers>
   <!-- comment everythnig else -->

     <server>
       <id>maven.oracle.com</id>
       <username>dalibor.blazevic@sysco.no</username>
       <password>{0KgcFKDly/EHMBbpakec7WAXzS7fazN0ll41W7PY5xI=}</password>
       <configuration>
         <basicAuthScope>
           <host>ANY</host>
           <port>ANY</port>
           <realm>OAM 11g</realm>
         </basicAuthScope>
         <httpConfiguration>
           <all>
             <params>
               <property>
                 <name>http.protocol.allow-circular-redirects</name>
                 <value>%b,true</value>
               </property>
             </params>
           </all>
         </httpConfiguration>
       </configuration>
     </server>

  </servers>
```

  b.) We have to replace <profiles> section with the content below that includes repository definition and location for Apache Maven Central and Oracle Maven repository.

```xml
  <profiles>
   <!-- comment everythnig else -->

   <profile>
     <id>main</id>
     <activation>
       <activeByDefault>true</activeByDefault>
     </activation>
     <repositories>
       <repository>
         <id>Maven Central</id>
         <name>Maven Central</name>
         <url>http://repo.maven.apache.org/maven2</url>
       </repository>
       <repository>
         <id>maven.oracle.com</id>
         <name>maven.oracle.com</name>
         <url>https://maven.oracle.com</url>
       </repository>
     </repositories>
   </profile>

  </profiles>
```

  c.) If we want to support deployment phase of Maven build and enable automatic deployment to WebLogic server, we have to include <properties> section with WebLogic server URL and credentials, as in following example:

```xml
  <profile>
   <id>main</id>
   <activation>
     <activeByDefault>true</activeByDefault>
   </activation>

   <pluginRepositories>
       <pluginRepository>
       <id>maven.oracle.com</id>
         <url>https://maven.oracle.com</url>
       </pluginRepository>
   </pluginRepositories>

   <properties>
       <oracleUsername>weblogic</oracleUsername>
       <oraclePassword>welcome1</oraclePassword>
       <oracleHome>/u01/app/Oracle/Middleware</oracleHome>
       <oracleServerUrl>http://localhost:7101</oracleServerUrl>
   </properties>

   <repositories>
     <repository>
       <id>maven.oracle.com</id>
       <name>maven.oracle.com</name>
       <url>https://maven.oracle.com</url>
     </repository>
   </repositories>
  </profile>
```

9.) Optionaly if you have not included <properties> in <profile> section you can

  add it to each pom.xml

```xml
  <properties>
   <oracleHome>/u01/app/Oracle/Middleware</oracleHome>
   <oracleServerUrl>http://localhost:7101</oracleServerUrl>
   <oracleUsername>weblogic</oracleUsername>
   <oraclePassword>welcome1</oraclePassword>
  </properties>
```

10.) Now we have to test connectivity and synchronize Oracle Maven repository with our local repository by running following command:

```sh
    mvn com.oracle.maven:oracle-maven-sync:12.2.1-0-0:push -DoracleHome=${ORACLE_HOME}
```

11.) Finally to update archetype catalog with Oracle specific archetypes used to create OSB application and projects from the command line we use following command:

```sh
   mvn archetype:crawl -Dcatalog=$HOME/.m2/archetype-catalog.xml
```

For more in detail readings on connecting your environment with Oracle Maven repository and encripting passwords here are two usefull links:

[http://docs.oracle.com/middleware/1213/core/MAVEN/config_maven_repo.htm#MAVEN9012](http://docs.oracle.com/middleware/1213/core/MAVEN/config_maven_repo.htm#MAVEN9012) [http://maven.apache.org/guides/mini/guide-encryption.html](http://maven.apache.org/guides/mini/guide-encryption.html)

## Enabling Oracle Maven repository in JDeveloper and checking effective pom.xml
When we have set up connectivity with Oracle Maven repository from command line, it is time to enable Oracle maven repository support in JDeveloper if not enabled by default. For that open 'Preferences' from Tools or JDeveloper (on Mac) menu and chose Maven->Repositories section.

![](/images/2016-04-21-HowTo-fix-Maven-in-OSB/01.png)

In that section first add reference to 'maven.oracle.com' repository by using the same ID that you have used in <properties> and <servers> section of settings.xml file (step 1. indicated on the image). Than add repository URL and Index URL (as indicated in step 2. on the image) and test connectivity. Then select all repositories and index them (step 3. and 4. from the immage). Finally index local repository (step 5. on the image).

If previous step is working we can check whether we can get effective pom.xml from both JDeveloper and command line. In JDeveloper open pom.xml file in editor and click 'Effective POM' tab at the bottom of the file:

![](/images/2016-04-21-HowTo-fix-Maven-in-OSB/02.png)

Still we have to check if we can get effective pom.xml from the command line as well. For that we have to go to the project directory where the project pom.xml file resides and execute:

```sh
 mvn -e -X help:effective-pom
```

#Building OSB projects with Maven

If we do not get any errors after testing effective pom.xml we are almost ready to try our first build. Unfortunately every time we create new project in JDeveloper the bogus ADF plugin is added to the project and pom.xml file looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.oracle.servicebus</groupId>
        <artifactId>sbar-project-common</artifactId>
        <version>12.1.3-0-0</version>
    </parent>

    <groupId>MavenTest3</groupId>
    <artifactId>SBProject2</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>sbar</packaging>

    <description/>
    <build>
        <plugins>
            <plugin>
                <groupId>com.oracle.adf.plugin</groupId>
                <artifactId>ojdeploy</artifactId>
                <version>12.2.1-0-0</version>
                <configuration>
                    <ojdeploy>
                        ${oracleHome}/jdeveloper/jdev/bin/ojdeploy
                    </ojdeploy>
                    <workspace>
                        ${basedir}/../MavenTest3.jws
                    </workspace>
                    <project>
                        SBProject2
                    </project>
                    <profile>
                        SBProject2
                    </profile>
                    <outputfile>
                        ${project.build.directory}/${project.build.finalName}.${project.packaging}
                    </outputfile>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>deploy</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    <properties>
        <oracleHome>
            ${env.ORACLE_HOME}
        </oracleHome>
    </properties>
</project>
```

The whole <build> and <properties> section needs to be thrown out from pom.xml file in order for it to work. When we have done that we are ready to test our first build by right clicking pom.xml file in Application window project section and choosing 'Run Maven'->package or using following command from the project directory containing pom.xml file:

```sh
 mvn -e -X package
```

Result is archive sbconfig.sbar that is saved in: <projec directory>/.data/maven directory. Although it is perfectly valid OSB archive and as is can be imported to OSB console, it is not practical that every project archive has a same name and that it is placed in temporary directory that eventually can be deleted. Especially if we have to upload project manually. Therefore we can use an additional plugin that will be used during the package phase to rename and move sbconfig.sbar archives to more suitable location. First we have to prepare location and as a rule of thumb it is best to create 'build' directory under the OSBApplication directory. That means in the same directory that is hosting all OSB projects belonging to the same application. After that we are ready to modify our pom.xml somewhat and it will look like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.oracle.servicebus</groupId>
        <artifactId>sbar-project-common</artifactId>
        <version>12.1.3-0-0</version>
    </parent>

    <groupId>MavenTest3</groupId>
    <artifactId>SBProject2</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>sbar</packaging>

    <description/>
    <build>
      <plugins>
        <plugin>
            <artifactId>maven-antrun-plugin</artifactId>
            <version>1.7</version>
            <executions>
            <execution>
              <phase>package</phase>
              <configuration>
                <target>
                 <copy file="${project.build.sourceDirectory}/.data/maven/sbconfig.sbar"
                 tofile="${project.build.outputDirectory}/${project.build.finalName}.jar"/>
                </target>
              </configuration>
              <goals>
                <goal>run</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
      </plugins>
        <outputDirectory>../build/</outputDirectory>
    </build>
</project>
```

Antrun plugin that we added, is going to copy sbconfig.sbar to prepared 'build' directory specified in <outputDirectory> section and it will rename it to SBProject2-1.0-SNAPSHOT.jar using project name and version specified in pom.xml file. Now we can test also Maven deployment phase by right clicking pom.xml file in Application window project section and choosing 'Run Maven'->deploy or using following command from the project directory containing pom.xml file:

```sh
 mvn -e -X deploy
```

[NOTE]: Unfortunately for us each time we modify project properties in JDeveloper the bogus ADF plugin is added again and we have to fix pom.xml file to look as explained above!

#OSB Application build and deployment

Very often we have to build and deploy several project at once as being part of the same application. For that to work we have to prepare Application pom.xml file and check that it has been correctly updated each and every time we add a new project. Application pom resides in the same directory containing project directories and can be opened by using 'Application Sources' accordion bar from the 'Application' window in Jdeveloper as can be seen from the picture below:

![](/images/2016-04-21-HowTo-fix-Maven-in-OSB/03.png)

The Application pom.xml should contain the list of projects belonging to the same application. The building and deploying of different projects will be executed in exactly the same order specified in pom.xml that looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>MavenTest3</groupId>
    <artifactId>MavenTest3</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>pom</packaging>

    <modules>
        <module>System</module>
        <module>SBProject2</module>
        <module>SBProject3</module>
        <module>SBProject4</module>
    </modules>

</project>
```

From within application directory we can use the same maven commands:

```sh
 mvn -e -X deploy
```

and

```sh
 mvn -e -X deploy
```

to package and/or deploy all projects at once.

#Conclusion

Unfortunately there is a lot of manual work that has to be done in preparing our OSB environment to support Maven builds and deployment. Most of these tasks are one time job only. However due to the bug in JDeveloper each and every time we create new project or update project properties we have to check project pom.xml file and application pom.xml file and correct them as described it this article. Once that all the preparation and checking has been done correctly we can run our automated integration and build seamlessly.
