---
layout: post
title: Publishing a Kotlin library to your Bintray repo using Gradle Kotlin DSL and Bintray Gradle Plugin
categories: OSS
tags: [kotlin,java,bintray,gradle,maven]
author: serpro
---

<!-- TODO img -->

*This post was originally posted on [Medium](https://serpro69.medium.com/publishing-a-kotlin-library-to-your-bintray-repo-using-gradle-kotlin-dsl-bdeaed54571a)*

If you’re working on an OSS project you will most likely come to a point where you will want to publish your artifacts to a publicly accessible repository (A Central Maven repository, for example, if you’re working on a java-based project.) For someone who has never published an artifact to be available to others the process can be somewhat unclear, and scarce documentation makes things even more daunting. If on top of that you’re using gradle with kotlin dsl build scripts then chances are documentation could be even sparser, even though the documentation improves constantly.

Until recently I have been working as a test automation engineer (now working in development) and have been coding for a living for some time now. Software testing and Quality Assurance are one of my passions at work and naturally my "OSS pet projects" are also related to this area. Some time ago I’ve gone through the process of publishing my first OSS project, a Kotlin library for generating fake data for tests: [kotlin-faker](https://github.com/serpro69/kotlin-faker). The process wasn't entirely clear at first, but once you understand the details of publishing your artifacts to a publicly available repository things become quite simple.

## Where to publish?

<!-- TODO img -->

At first, I was thinking to publish everything directly to [Maven Central](https://search.maven.org/classic/). This seems like a natural choice since this is one of the most commonly used repos out there and is directly accessible from most Maven and Gradle builds without the need to specify any additional information such as the url to your custom repo.

However, there are some caveats to doing this, and I would generally recommend using a custom repository, for example Bintray (which is what I am using myself and will be using as an example in this article) unless you’re absolutely sure what you’re doing.

#### Why is Bintray preferable to Maven Central (for me)?

First, with Maven Central you have to be sure about your setup. You can play around with your publishing setup and always undo things in Bintray, but not in Maven Central. Second is that you have complete control over your files and if you’re not sure about the quality of your library and want to test things — this would be a good place to do this. In Maven Central you sort of loose all control after you publish something there. Last, Maven Central has more strict requirements when it comes to publishing artifacts. 

*Note: while you do have the ability to modify your files in your own Bintray repo, you should be careful while doing so as this might break things for people who are already using these artifacts.*

After publishing to Bintray and verifying that everything is as you want it to be you can then sync your artifacts JCenter and to Maven Central directly from the UI, through the bintray plugin configuration or even the APIs. You can read more about it in their [blogpost](https://blog.bintray.com/2014/02/11/bintray-as-pain-free-gateway-to-maven-central/).

## The Setup

### Prerequisites

These are pretty self explanatory and easy to find, so I won't go into too many details here and will just list what you will need to do.
* Create a new account (you can also use your existing github, google, or twitter accounts for authentication)
* Add a new repository (Select the type as ‘Maven’.) - this is the repository where you will publish your packages, and from where others will be able to download them.
* Get your API key — go to ‘edit profile’, and you will see the ‘API Key’ tab there.

### Gradle build file configuration

First we need to add the `maven-publish` and `bintray` plugins added to the `gradle.build.kts` build script file:
```
plugins {
    `maven-publish`
    id("com.jfrog.bintray") version "1.8.4"
}
```

#### Configuring maven-publish plugin

After adding `maven-publush` plugin you should have a `publishing` extension function available, which is used to configure your publication:

```
publishing {
    publications {
        create<MavenPublication>(publicationName) {
            groupId = artifactGroup
            artifactId = artifactName
            version = artifactVersion
            from(components["java"])

            pom.withXml {
                asNode().apply {
                    appendNode("description", pomDesc)
                    appendNode("name", rootProject.name)
                    appendNode("url", pomUrl)
                    appendNode("licenses").appendNode("license").apply {
                        appendNode("name", pomLicenseName)
                        appendNode("url", pomLicenseUrl)
                        appendNode("distribution", pomLicenseDist)
                    }
                    appendNode("developers").appendNode("developer").apply {
                        appendNode("id", pomDeveloperId)
                        appendNode("name", pomDeveloperName)
                    }
                    appendNode("scm").apply {
                        appendNode("url", pomScmUrl)
                    }
                }
            }
        }
    }
}
```

Notice the `pom.withXml` part. We need to set certain metadata before publishing a library. Maven Central, for example, has some [requirements](https://central.sonatype.org/pages/requirements.html#sufficient-metadata) on what metadata needs to be present in the pom.

This can be done as seen in code above. Another way is to utilize the DSL that Maven Publish Plugin provides for this purpose:

```
pom {
    name = 'My Library'
    description = 'A concise description of my library'
    url = 'http://www.example.com/library'
    properties = [myProp: "value", "prop.with.dots": "anotherValue"]
    licenses {
        license {
            name = 'The Apache License, Version 2.0'
            url = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
        }
    }
    developers {
        developer {
            id = 'johnd'
            name = 'John Doe'
            email = 'john.doe@example.com'
        }
    }
    scm {
        connection = 'scm:git:git://example.com/my-library.git'
        developerConnection = 'scm:git:ssh://example.com/my-library.git'
        url = 'http://example.com/my-library/'
    }
}
```

*Note: it is not necessary to explicitly specify the `pom` metadata - it will be generated by the plugin automatically, therefore the `pom {}` configuration can be omitted altogether unless you want to set/modify some things in the generated pom explicitly.*

Usually you want to publish javadocs and sources as well. To create sources.jar we need to create a task that will generate a jar:

```
val sourcesJar by tasks.creating(Jar::class) {
    archiveClassifier.set("sources")
    from(sourceSets.getByName("main").allSource)
}
```

And then add it to the `publishing` configuration:
```
publishing {
    publications {
        create<MavenPublication>(publicationName) {
            artifact(sourcesJar)
        }
    }
}
```

To generate kdocs (javadoc in Kotlin world) you should use the [Kotin/dokka](https://github.com/Kotlin/dokka) plugin.

For more info on using Maven Publish Plugin also see the [official userguide](https://docs.gradle.org/current/userguide/publishing_maven.html) page.

#### Configuring Bintray plugin

The Bintray plugin takes care of actually uploading artifacts to your repository.

A basic setup would look something like this:
```
bintray {
    user = project.findProperty("bintrayUser").toString()
    key = project.findProperty("bintrayKey").toString()

    publish = true // Set to false if you don't want to automatically publish the release after uploading the artifacts

    setPublications(publicationName) // We're using the publication created by the `maven-publish` plugin

    pkg.apply {
        repo = "maven" // This is the name of the repo that you have previously created in Bintray
        name = artifactName
        userOrg = user
        githubRepo = githubRepo
        vcsUrl = pomScmUrl
        description = "Some description can go here, but is optional"
        setLabels("tag1", "tag2", "tag3")
        setLicenses("MIT")
        desc = description
        websiteUrl = pomUrl
        issueTrackerUrl = pomIssueUrl
        githubReleaseNotesFile = githubReadme

        version.apply {
            name = artifactVersion
            desc = pomDesc
            released = Date().toString()
            vcsTag = artifactVersion
        }
    }
}
```

A few things to keep in mind:
* the repo should match with whatever repository name you specified when creating a new repo on Bintray. I simply named it `maven`, but it can be named any way you want.
* the user and key creation were covered in the **Prerequisites** section of this post. You will need to pass them when uploading the artifacts, i.e. from the command line or from the CI tool.

The rest is pretty much self explanatory, but I've added a few comments in the code just in case.

## Publishing to your Bintray repo

When you're ready to upload a new release just the command: `$ ./gradlew bintrayUpload -PbintrayUser="my_bintray_username" -PbintrayKey="my_bintray_api_key"`

This can also be set up with CI, where `bintrayUser` and `bintrayKey` properties will be stored as "secrets".

## Downloading artifacts from your repository

In order to be able to use the published package as a dependency in a build file we first need to add our repository to the list of repositories. In gradle it would look something like this:
```
repositories {
    maven {
        setUrl("https://dl.bintray.com/serpro69/maven/")
    }
}
```

After that the library can be added as a usual dependency:
```
dependencies {
    implementation '$groupId:$artifactId:$version'
}
```

## Retrospective

All in all this wasn't actually that hard. I needed to collect some pieces of missing information here and there and understand what’s happening when you publish an artifact and what are the prerequisites for that. After that the process looks quite straightforward.

Gradle is also way superior to Maven when it comes to build configurations, but this shouldn't be a surprise to anyone nowadays.

Having a personal repository on Bintray where you can actually play around and have control over your files helps a lot. I’m glad I didn't go with Maven Central straight away.

You can also check out the complete build file of the project I mentioned in the beginning [here](https://github.com/serpro69/kotlin-faker/blob/master/core/build.gradle.kts). Hopefully it will help you move in the right direction when publishing your artifacts to Bintray.
