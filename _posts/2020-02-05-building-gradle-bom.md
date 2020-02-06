---
layout: post
title: Creating Maven like BOM in Gradle Kotlin DSL with full versioning support for plugins and dependencies
categories: CI-CD
tags: [Maven, Gradle, Kotlin]
author: dalibor
---

# Introduction

Most of developers and DevOps engineer that are switching from Maven dependency management and Build tool, are familiar with the concept of Maven "Bill Of Material".They find very soon that creating the same thing in Gradle is nowhere as easy as it was in Maven. When they start to dig into Gradle documentation on this topic they pretty soon are realising that documentation is scattered all over the place lacking an easy explanation and guidance on how to build Maven like BOM in Gradle.

The aim of this tutorial is to fill in this gap an to provide an easy to understand guide on why and how to build Maven like BOM in Kotlin Gradle DSL. Even step further it ads capability to provide centralised management of plugin and dependency versions.

# Bring it On Bill

On any major project we are often dealing with the problem of coordinating and managing many different and largely independent builds. Especially is that present in Microservice world. Each Microservice is mini project on its own, with his own GIT repository, and its own set of dependencies and plugins required to perform the build. This seams not to be a problem before we try to build all the projects on common build server and doing some code analysis on the fly. Pretty soon code analyser will throw report with tons of outdated packages with security vulnerabilities, our internal repository manager is soon cluttered with dozens of versions of the same package and some strange bugs are emerging due to the fact that internal package has been updated while our Microservice is still depending on the older version. We are not even spotting the problem until Microservice is already in production

Solution to above mentioned problems is to have the common centralised place where version of all build plugins and dependencies can be managed. All of the build projects are then referring to this centralised place. Moreover this centralised place can hold definition to some common plugins, tasks and repositories that all of the referring projects are using. This place is usually referred to as "Bill Of Material" or BOM for short.

## Maven BOM example

Maven build system is purely declarative. That means that whole build process is defined as a set of statements or declarations (no code). In the Maven case XML syntax is used to specify declarations. So in this light let us see how the Maven BOM looks like:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>no.sysco</groupId>
    <artifactId>base</artifactId>
    <packaging>pom</packaging>

    <name>base</name>
    <version>1.0-SNAPSHOT</version>

    <distributionManagement>
        <snapshotRepository>
            <id>snapshots</id>
            <url>http://localhost:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>

    <properties>        
        <apache.cxf.version>3.2.6</apache.cxf.version>        
        <allure.maven.version>2.9</allure.maven.version>
        <commons-beanutils.version>1.9.3</commons-beanutils.version>
        <commons-io.version>2.6</commons-io.version>        
        <jdk.version>1.8</jdk.version>        
        <maven.plugins.clean.version>3.1.0</maven.plugins.clean.version>
        <maven.plugins.compiler.version>3.8.0</maven.plugins.compiler.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>        
        <slf4j.log4j-over-slf4j.version>1.7.25</slf4j.log4j-over-slf4j.version>
        <slf4j.slf4j-simple.version>1.7.5</slf4j.slf4j-simple.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>commons-beanutils</groupId>
                <artifactId>commons-beanutils</artifactId>
                <version>${commons-beanutils.version}</version>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>${commons-io.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>    
      <dependency>
          <groupId>org.slf4j</groupId>
          <artifactId>log4j-over-slf4j</artifactId>
          <version>${slf4j.log4j-over-slf4j.version}</version>
      </dependency>
      <dependency>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-simple</artifactId>
          <version>${slf4j.slf4j-simple.version}</version>
      </dependency>
   </dependencies>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>io.qameta.allure</groupId>
                    <artifactId>allure-maven</artifactId>
                    <version>${allure.maven.version}</version>
                </plugin>
                <plugin>
                    <groupId>org.apache.cxf</groupId>
                    <artifactId>cxf-codegen-plugin</artifactId>
                    <version>${apache.cxf.version}</version>
                </plugin>
            </plugins>
        </pluginManagement>

        <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-clean-plugin</artifactId>
              <version>${maven.plugins.clean.version}</version>
          </plugin>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-compiler-plugin</artifactId>
              <version>${maven.plugins.compiler.version}</version>
              <configuration>
                  <source>${jdk.version}</source>
                  <target>${jdk.version}</target>
                  <encoding>${project.build.sourceEncoding}</encoding>
              </configuration>
          </plugin>
        </plugins>
    </build>

</project>
```

To quickly recap, let us explain different elements in Maven BOM. This will enable us to understand Gradle counterpart little bit easier:

- `<distributionManagement>` section describes to which repository this BOM is going to be deployed to
- `<properties>` section lists versions of all dependencies, plugins, constraints and configurations used later in the BOM file
- `<dependencyManagement>` section list all dependencies that potentially are going to be used in projects referencing this BOM
- `<dependencies>` section that resides outside `<dependencyManagement>` section list all dependencies with respective versions, that are common to all projects referencing this BOM
- `<pluginManagement>`section list all plugins that potentially are going be used in projects referencing this BOM
- `<plugins>` section that resides outside `<pluginManagement>` section list all plugins with respective versions, that are common to all projects referencing this BOM

From this example we can se that Maven BOM provide us with easy to manage centralised place where we can specify common build things (plugins and dependencies) as well as versions for all dependencies and plugins that potentially are going be used in projects referencing this BOM

# Let us Gradle the code

(Un)fortunately Gradle build script is code. Not set of declarative statements. This give us more flexibility, especially when it comes to building complex multilayered projects. However this adds to complexity when creating our builds also. Gradle syntax is quit strict when it comes to ordering of various elements in the script. One such example is a `plugins {}` section that has to be the first statement in each build script. This is quite logical as Gradle interpreter does not know nothing on its own unless we tell it what to do. So anything Gralde can do is by means of various plugins providing us with various built-in tasks. Some tasks are independent, some are dependent on other tasks. Some tasks can be specialized and some extended. One problem that this creates is that all plugins and all plugin versions have to be specified at the beginning of each `gradle.build` or `gradle.build.kts` file. Depending wheter we are using Groovy or Kotlin as Gradle DSL or scripting language. So here is Kotlin example for `plugins {}` section:

```kotlin
plugins {
  kotlin("jvm") version "1.3.20"
  kotlin("plugin.spring") version "1.3.20"
  id("com.github.ManifestClasspath") version "0.1.0-RELEASE"
  id("maven-publish")
}
```

Later on we will se how to work around this problem, but let us now focus on creating Maven like BOM to start with, where we are going to specify dependencies and respective versions. Typical Gradle block specifying dependencies looks like this:

```kotlin
dependencies {
  //Spring boot dependencies
  implementation("org.springframework.boot:spring-boot-starter-actuator:$2.2.1.RELEASE")
  implementation("org.springframework.boot:spring-boot-starter-web:2.2.1.RELEASE") {
    exclude(module = "spring-boot-starter-tomcat")
  }

  testImplementation("com.jayway.awaitility:awaitility:1.7.0")
  testImplementation("com.github.tomakehurst:wiremock-standalone:2.15.0")

  testRuntimeOnly("org.junit.jupiter:junit-jupiter-engine")
}
```

## BOM the Gradle

What we don't want is that every Microservice project has his own version of the same dependencies and this version are usually not maintained and far from being on last stable version that can be found in [Maven Central](https://mvnrepository.com/repos/central) for example. So we will build the common Gradle build project that all other projects can refer to. So let us do it. First we need to specialised plugins in our build script:

```kotlin
plugins {
  `java-platform`
  `maven-publish`
}
```

- `java-platform` plugin is used to build Maven like BOM
- `maven-publish` plugin is used to publish generated Maven like BOM XML file to desired repository

Now we can specify our dependencies that are going to be mapped to Maven BOM `<dependencyManagement>` section:

```kotlin
dependencies {
  //Maven BOM <dependencyManagement> block
  constraints {
    api("commons-httpclient:commons-httpclient:3.1")
    runtime("org.postgresql:postgresql:42.2.5")
  }
}
```

This effectively maps to:

```xml
<dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>commons-httpclient</groupId>
        <artifactId>commons-httpclient</artifactId>
        <version>3.1</version>
      </dependency>
      <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.2.5</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
```

We can optionally include common dependencies that are going to be applied to all projects using this BOM. To do this we have to specifically enable this feature:

```kotlin
javaPlatform {
  allowDependencies()
}
```

Now we can use it:

```kotlin
dependencies {
  val jacksonModuleVersion: String by project
  api("com.fasterxml.jackson.module:jackson-module-kotlin:$jacksonModuleVersion")
}
```

This effectively maps to:

```xml
<dependencies>
  <dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-kotlin</artifactId>
    <version>2.9.6</version>
    <scope>compile</scope>
  </dependency>
</dependencies>
```

As we can see from the above example, dependency version can be hardcoded in the file or provided as property in `gradle.property` file, like `jacksonModuleVersion=2.9.6` property that is mapped to string variable trough delegate object.

We can also import BOM from another BOM creating hierarchy of BOMs:

```kotlin
dependencies {
  api(platform("com.fasterxml.jackson:jackson-bom:2.9.8"))
}
```

This maps to:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.fasterxml.jackson</groupId>
      <artifactId>jackson-bom</artifactId>
      <version>2.9.8</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

Finally we want to publish our generated BOM to Maven compatible repository. We do that using `publishing {}` section:

```kotlin
publishing {
  repositories {
    maven{
      //Properties from ./gradle/gradle.properties to variable mapping
      val localMavenUser: String by extra
      val localMavenPassword: String by extra

      //Publishing Maven repository URL and credentials
      url = uri("http://localhost:8081/repository/maven-snapshots/")
      credentials {
        username = localMavenUser
          password = localMavenPassword
      }
    }
  }

  //Defines Maven BOM as valid publication
  publications {
    create<MavenPublication>("myPlatform") {
      from(components["javaPlatform"])
    }
  }
}
```

- `publications {}` section defines that generated BOM will be produced as `pom-default.xml` found in `build/publications/myPlatform` folder.
- `repositories {}` section defines where this generated BOM will be send to. Credential can be supplied from properties file that is usually located in: `./gradle/gradle.properties`

Full `build.gradle.kts` code can be found in: <https://github.com/cubeprogramming/GradleBOM/blob/master/MavenBOMPublisher/build.gradle.kts>

## Let's BOM it

Now when we have defined and deployed BOM we have to reference it in our project `build.gradle.kts` file. So first we need to define the location where we can get it. We do this by using `repositories {}` block and referencing our internal Maven compatible repository:

```kotlin
repositories {
    mavenCentral()
    maven(url = uri("http://localhost:8081/repository/maven-snapshots/"))
}
```

Though we have to import our BOM as a dependency in order to use it:

```kotlin
dependencies {
    //api(platform(project("JavaPlatformProject")))
    implementation(platform("com.cubeprogramming:MavenBOMPublisher:1.0-SNAPSHOT"))
}
```

Now we are able to specify dependencies without specify version:

```kotlin
dependencies {
    //api(platform(project("JavaPlatformProject")))
    implementation(platform("com.cubeprogramming:MavenBOMPublisher:1.0-SNAPSHOT"))
    implementation("stdlib-jdk8")
    testImplementation("junit", "junit", "4.12")
}
```

However nothing prevent us from overiding versions of referred depenencies or introduce new one if we need to to so.

## How about plugins?

As said previously versioning plugins is little bit tricky as `plugins {}` section is always first section in every build script and generally does not allow specifying plugins outside of this section. Specifying plugin versions is a must. Only exception to this rule are some Gradle built-in plugins like `java`, `java-platform` or `maven-publish` plugin for example. as they are inheriting version numbers from [gradlePluginPortal()](https://plugins.gradle.org/search?term=central) which is default built in location for Gradle plugins and thus does not have to be explicitly specified.

So how to work around this problem? Fortunately for us, from Gradle 6.x it is possible to use `settings.gradle.kts` file to move some of the logic from `build.gradle.kts` to it. This allow us to build one common `settings.gradle.kts` that every project can implement in the same way. Essentially copy and paste.

### Special `settings` file

If we want to have common set of settings for every project we build and we are not prepared to create [Gradle multi-project build setup](https://docs.gradle.org/current/userguide/multi_project_builds.html#multi_project_builds), than the only solution is to create common `settings.gradle.kts` file that all project will copy to root project folder, along with the project specific `build.gradle.kts`. Let us see how the file looks like:

```kotlin
pluginManagement {
    //Defining access to remote gradle.properties file that resides on GIT server
    val prop = java.util.Properties()
    val propertyResource = java.net.URL("https://raw.githubusercontent.com/cubeprogramming/GradleBOM/master/MavenBOMPublisher/gradle.properties")
    prop.load(java.io.InputStreamReader(propertyResource.openStream()))

    //Defining access to plugins.properties file that resides on GIT server
    val pluginsProp = java.util.Properties()
    val pluginsPropResource = java.net.URL("https://raw.githubusercontent.com/cubeprogramming/GradleBOM/master/MavenBOMPublisher/plugins.properties")
    pluginsProp.load(java.io.InputStreamReader(pluginsPropResource.openStream()))

    plugins {
        java
        kotlin("jvm") version prop.getProperty("kotlinPluginVersion")
        pluginsProp.forEach{
            (k, v) -> id(k.toString()) version v.toString()
        }
    }
}

gradle.allprojects{
    apply(plugin = "java")

    repositories {
        //Defining access to repostoryList file that resides on GIT server
        val repositoryListRes = java.net.URL("https://raw.githubusercontent.com/cubeprogramming/GradleBOM/master/MavenBOMPublisher/repositoryList")
        val repositoryLines = java.io.BufferedReader(java.io.InputStreamReader(repositoryListRes.openStream()))

        mavenCentral()
        repositoryLines.forEachLine {
            maven(url = uri(it))
        }
    }

    configure<JavaPluginConvention> {
        sourceCompatibility = JavaVersion.VERSION_1_8
    }
}
```

The first section in the file `pluginManagement {}` corresponds with `<pluginManagement>` section in the typical Maven BOM. However, as this `settings.gradle.kts` is going to be copied to each project we still does not have centralised management of plugin versions unless we do some trick. The trick is to load the plugin names and versions from the property file that is accessible from every project. The best place to put this file is on the same GIT server that all other projects, that we are managing, are using. Obvious choice would be to put the file in the same project that is generating Maven BOM with `<dependencyManagement>` section. So we have created [plugins.properties](https://github.com/cubeprogramming/GradleBOM/blob/master/MavenBOMPublisher/plugins.properties) file in that project.

The next section in `settings.gradle.kts` file is `gradle.allprojects {}`. This section allow us to apply common configuration to all projects. Again we would like to load as much as possible of configuration settings from the central place. So as before have we decided to put this file is on the same GIT server that all other projects, that we are managing, are using. That is the same project that is generating Maven BOM.

This [repositoryList](https://github.com/cubeprogramming/GradleBOM/blob/master/MavenBOMPublisher/repositoryList) file contains the list of common repositories that are going to be used to download plugins and dependencies. List is than loaded from the server and applied in the `repositories {}` section.

Now our [build.gradle.kts](https://github.com/cubeprogramming/GradleBOM/blob/master/BOMConsumingProject/build.gradle.kts) looks pretty simple:

```kotlin
plugins {
    kotlin("jvm")
    id("io.spring.dependency-management")
}

group = "com.cubeprogramming"
version = "1.0-SNAPSHOT"


dependencies {
    implementation(platform("com.cubeprogramming:MavenBOMPublisher:1.0-SNAPSHOT"))
    implementation(kotlin("stdlib-jdk8"))
    testImplementation("junit", "junit", "4.12")
}

tasks {
    compileKotlin {
        kotlinOptions.jvmTarget = "1.8"
    }
    compileTestKotlin {
        kotlinOptions.jvmTarget = "1.8"
    }
}
```

# Conclusion

As we can se from simplified [build.gradle.kts](https://github.com/cubeprogramming/GradleBOM/blob/master/BOMConsumingProject/build.gradle.kts), there is no need to define and configure `java` plugin as it is already defined and configured in `settings.gradle.kts`. Also plugin versions are not specified as versions specification is downloaded from the GIT server. No need for `repositories {}` section as repository list is also being downloaded from the GIT server. And finally, in our `dependencies {}` section we are importing generated Maven BOM from our internal Maven repository server, thus allowing us to specify dependencies, without specifying versions.

All our repository, plugin and dependency definition and versions are now managed centrally, thus reducing possibility that different project are using different plugin/dependency version for the same type of plugin/dependency. Centralised management allow us to reduce copying of boilerplate code as well.

The complete project example can be accessed on: <https://github.com/cubeprogramming/GradleBOM>
