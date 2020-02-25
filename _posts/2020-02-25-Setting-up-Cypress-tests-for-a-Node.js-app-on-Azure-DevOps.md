---
layout: post
title: Setting up Cypress tests for a ode.js app on Azure DevOps
categories: testing
tags: [testing,azure,devops,node.js,cypress]
author: AnitaLipsky 
---

# Introduction
This shows how to set up Cypress([cypress.io](cypress.io)) tests towards a Node.js application on an Azure DevOps pipeline, based on how the team did this for the first time, step by step.

The steps were done using shared team knowledge.


# Overview of steps
- Goal 1: Run Cypress test towards a Node.js app
- Goal 2: Run Cypress tests on an Azure DevOps pipeline
- Goal 3: Send test results to a Cypress Dashboard
- Goal 4: Configure pipeline to fail if tests fail

# Goal 1: Run Cypress test towards a Node.js app
## Background: Selection of Cypress as the test framework

Cypress was the test automation framework chosen for this project due to it seeming like a good match for the project based on a previous trial, seeing this used successfully for another project, and experience with other test frameworks.

The project is a 20 week, 10 sprint delivery for revamping web application used for both back office administration and customer facing administration. The database will be migrated to the cloud, an API will be built and a new GUI will be created. The team consists of an architect, backend / middleware developers, a front end developer, a tester (myself) and project management.

## Try a quick Cypress pilot locally

The front end developer created a Vue app and some Cypress tests.

The tester (myself) setup several different types of Cypress tests to test different elements in a page for a standard web application, based upon the  Cypress getting started manual.

In this way we did not depend upon each other and could share different learnings.

We them demo’d this to each other, and were keen to continue with Cypress.

## Commands for running the tests on the command line in the node application

Here are the commands for running the tests in UI and headless mode.

UI mode is how the tests will be run locally when building and testing new
features:
```bash
npm run test:e2e
```

Headless mode is how the tests will be run on the pipeline:
```bash
npm run test:e2ehl
```

Add the shortcuts “<b>test:e2e</b>” and “<b>test:e2ehl</b>” to package.json

Snip:
```json
{
 "name": "app-ui",
 "version": "0.1.0",
 "private": true,
 "scripts": {
 "serve": "vue-cli-service serve",
 "build": "vue-cli-service build",
 "test:unit": "vue-cli-service test:unit",
<b> "test:e2e": "vue-cli-service test:e2e",</b>
<b> "test:e2ehl": "vue-cli-service test:e2e --headless",</b>
 "lint": "vue-cli-service lint"
 }
```

Test the shortcut commands locally.

Run tests in UI mode locally, with shortcut:
```bash
npm run test:e2e
```

Run tests in headless mode locally, with shortcut:
```bash
npm run test:e2ehl
```

# Goal 2: Run Cypress tests on an Azure DevOps pipeline

Pre-requisite: The Azure DevOps space was set up with the UI repository

In short, create and configure a pipeline to run the tests in headless mode

## Create pipeline
In Azure DevOps
* *Create pipeline > new pipeline > select Azure > select the repository > select “Node.js with Vue” > create branch and commit message with appropriate links to work item
* The pipeline will start running, take a peek and follow along with the steps to see what it does out of the box

## Configure pipeline to run the tests in headless mode

Update azure-pipelines.yml to run the tests in headless mode:

Snip azure-pipelines.yml :
```yaml
# Node.js with Vue

# Build a Node.js project that uses Vue.

# Add steps that analyze code, save build artifacts, deploy, and more:

# https://docs.microsoft.com/azure/devops/pipelines/languages/javascript

trigger:
 - master

pool:
 vmImage: 'ubuntu-latest'

steps:
 - task: NodeTool@0
   inputs:
     versionSpec: '10.x'
   displayName: 'Install Node.js'

 - script: |
    npm install
    <b>npm run test:e2ehl</b>
    npm run build
   displayName: 'npm install, test and build'

After pushing the update to the repository, verify the logs to be sure the test ran.

Snip:
```
<b> My First Test
✓ Visits the app root url (982ms)</b>
```

# Goal 3: Send test results to a Cypress Dashboard
Cypress has a Dashboard feature which shows test run results in a simple an
elegant way.

## Pre-requisites
* A Cypress dashboard was created and tested locally
* It was then shared with some team members for testing
* Note that Cypress Dashboard is a paid feature depending on usage

## Update package.json with the new command to send test results to the dashboard
Snip:
```json
{…
 "build": "vue-cli-service build",
 "test:unit": "vue-cli-service test:unit",
 "test:e2e": "vue-cli-service test:e2e",
 "test:e2ehl": "vue-cli-service test:e2e --headless",
<b> "test:e2e-pipeline": "vue-cli-service test:e2e --headless --record --key the-key-goes-in-here",</B<
 "lint": "vue-cli-service lint"
 }
```

## Update azure-pipelines.yml to run this command instead
Snip:
```
 - script: |
     npm install
     <b>npm run test:e2e-pipeline</b>
     npm run build
```

Now when the pipeline is run it is possible to click through to the Dashboard.
For example, clicking on a result shows links to test test output, a video of the test run in the case of failure, and so on.

# Goal 4: Configure pipeline to fail if tests fail

Currently the azure-pipeline.yml has install, test and build in one step which is a quick proof of concept of tests running, however this results in the pipeline showing success even if Cypress tests fail.

Therefore splitting up the steps is needed as follows.

Snip of azure-pipeline.yml:
```yaml
 - task: Npm@1
   inputs:
     command: 'install'
 - task: Npm@1
   inputs:
     command: 'custom'
     customCommand: 'run test:e2e-pipeline'
   displayName: 'Test'
 - task: Npm@1
   inputs:
     command: 'custom'
     customCommand: 'run build'
   displayName: 'Build'
```

Trigger the pipeline where there is one known failing test. If one does not exist, write one for the purpose of testing the pipeline.
The pipeline now shows this error message:


The full azure-pipeline.yml is now
```yaml
# Node.js with Vue
# Build a Node.js project that uses Vue.
# Add steps that analyze code, save build artifacts, deploy, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/javascript
trigger:
    - dev
    - master
    
pool:
  vmImage: 'ubuntu-latest'
  
steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '10.x'
    displayName: 'Install Node.js'
  
  - task: Npm@1
    inputs:
      command: 'install'
  - task: Npm@1
    inputs:
      command: 'custom'
      customCommand: 'run test:e2e-pipeline'
    displayName: 'Test'
  - task: Npm@1
    inputs:
 command: 'custom'
 customCommand: 'run build'
    displayName: 'Build'
```

A keen reader will note that the tests are also now triggered on both dev and master branches, instead of just master as in the initial steps, because the pipeline evolves along with the project.

# Conclusion

So far Cypress, Node.js with Vue and the Azure DevOps pipeline are working well together.

In addition, the tools are a good match for the project.

Finally, the main success factor has been a great team working together to share knowledge and experience.

That’s it - credit goes to the team for this setup!
