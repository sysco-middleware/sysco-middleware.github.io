---
layout: post
title: npm VS npx when running Cypress tests
categories: testing
tags: [testing,cypress,npm,npx]
author: AnitaLipsky 
---

# npm VS npx when running Cypress tests

This blog post is about something I stumbled over when figuring out which
commands to use for running [Cypress.io](http://cypress.io) tests when setting up a pipeline, in this case Azure DevOps. The pipeline in this post refers to configuring Azure
DevOps for running Cypress tests.

This post will run through a use case that leads to the following conclusions:
1. If using **npx**, beware the differences between **npm** and update the npx
command to use the same version the npm package uses
2. Add a handy generic *npm script command* to package.json that can be
used when trying out new *npm script commands*

Note: [Cypress.io](http://Cypress.io) is a test framework that I love using to automate UI tests mainly because of its interactive playback facility after a test run.


## What is npm?
Npm is the node package manager that comes shipped with node.js ([https://nodejs.org/en](https://nodejs.org/en))


## Where does npx come from?
Now npx is shipped together with node.js in the same way as npm. In other
words if node.js is installed, so is npx (and npm).


## What does npx do and how does it operate on Cypress?
Cypress is published as a executable program which makes it possible to type
on the command line
```bash
cypress run --headless
```

When a program is published as an executable program like this it is published
as a *binary*.

To run Cypress like this, normally you need to install Cypress first by using the
"npm install" command.

**npx** on the other hand is a tool which makes it possible to execute a binary, in
this case Cypress, without having to install Cypress first by using the "npm
install" command.

In other words, these are two different ways of running a binary:
* as a dependency of our project, using the npm tools for that purpose
* as an ad-hoc application, using npx, useful for one-off commands and exploratory stuff

## *npm script commands* for installing and running Cypress tests
We are using *npm script commands*, that is **npm run &lt;script&gt;** ([https://docs.npmjs.com/cli/run-script.html](https://docs.npmjs.com/cli/run-script.html)) to run Cypress tests on the pipeline

In order for an *npm script command* to work, the command must be configured inside the scripts section in package.json

For example
```bash
npm run test:e2ehl
```

requires configuring
```bash
test:e2ehl
```

in the scripts section of package.json

Example:
```bash
 "scripts": {
 "test:e2ehl": "cypress run --headless"
 }
 ```

Therefore when trying out a new *npm script command* you will need to modify
package.json unless there is a handy “generic” *npm script command* that you
can build upon. More on that later.

Note: the colon in “test:e2ehl” has no special semantic meaning, it's just there
to tell tests apart. In this case, it is the *npm script command* to run tests in
headless mode. Typically there will be multiple *npm script commands* each to
run tests in its own way, eg in UI mode, towards different servers, and so on.

## Why update *npm script commands*
As projects evolve, so does the test automation and build pipeline, and in this
case we were starting to set up tests to run on the pipeline in a new
configuration.

This project already had tests running on the pipeline using npm script
commands. However the current *npm script commands* were not compatible
with the new configuration we wanted to try.

## Why use npx commands
***npx*** commands do not require package.json to be updated and saved before
trying out each new command, which makes ***npx*** a quick start way to try out
new commands. This is unlike *npm script commands* which requires
package.json to be updated and saved before trying each new *npm script
command*.

When configuring a pipeline to run a new *npm script command*, this would
mean committing the updated command in package.json, creating a branch and
PR (pull request), merging the PR which builds and already triggers tests and
steps. This is too many steps to wait for when trying out new *npm script
commands* that run after existing steps.

Therefore I tried **npx** ([https://www.npmjs.com/package/npx](https://www.npmjs.com/package/npx) ) to try out new test configurations on the pipeline.

**Npx** is widely used and mentioned as an example in the Cypress *How to run
commands* [https://docs.cypress.io/guides/guides/command-line.html#How-torun-commands](https://docs.cypress.io/guides/guides/command-line.html#How-torun-commands)
```bash
npx cypress run
```

and further examples in the documentation use **npx**

In fact, **npx** is great if you haven’t yet downloaded Cypress and want to give
Cypress a quick try. It allows trying out some basic test examples without
having to think about adding various *npm script commands* to package.json or
even installing Cypress.

## npx downloads and use the latest version of the package specified
It does in some cases. In our case **npx** was downloading the latest version of
Cypress to run the tests towards.

The resulting behaviour was the Cypress tests were running slower than usual
and resulted in at least one consistent failure. Overall, the behaviour of the
tests were odd. I even wondered if there was a memory leak as for each test
run the tests seemed to get slower and slower until they timed out.

It turned out around the time we were trying out new test configurations using
**npx**, Cypress had just released a new major version and the tests were
behaving differently because we had a version that was quite a bit older. In
addition we were trying a new pipeline configuration so it was not entirely clear
to me the issues were to do with Cypress.

Since **npx** was downloading Cypress first before each test run this made
troubleshooting slow and used build resources we did not want to use.

## Solution: Update the npx command to use the same version the npm package uses
When **npx** downloads the latest version of the package specified, one solution
is to explicitly specify the version of the package, in this case Cypress, in order to ensure trying out new **npx** commands are running the same version as the *npm script commands*.

So for example instead of trying out an **npx** command
```bash
npx cypress run
```

which can lead to first downloading then running eg Cypress v5.0…

… update the command to specify the specific package version that corresponds to the current npm package version, eg
```bash
npx cypress@3.8.3 run
```

This is what I overlooked when trying out the new command.

This is not the full command - it is a snip relevant to this blog post.

## Better long term solution: Add handy “generic” *npm script command* that you can build upon
A better long term solution is to add a handy “generic” *npm script command* to
package.json that can be built upon on the command line.

In fact, the Cypress docs give an example of this in [https://docs.cypress.io/guides/guides/command-line.html#How-to-run-commands](https://docs.cypress.io/guides/guides/command-line.html#How-to-run-commands)

Example:

Update scripts in package.json to include
```bash
{
 "scripts": {
 "cy:run": "cypress run"
 }
}
```

… then when using npm pass the command’s arguments using the -- string
followed by the options
```bash
npm run cy:run -- --headless --record --key ourdashboard-record-key --tag 'fromNewPipelineStep --> toAppRunningOnTestServer'
```

This allows trying out new *npm script commands* on the command line without
having to update package.json

Once the *npm script command* is working as desired it can be added to
package.json scripts:
```bash
 "scripts": {
 "cy:run": "cypress run",
 "test:e2e-pipeline:testURL": "cypress run --headless
--record --key our-dashboard-record-key --tag
'fromNewPipelineStep --> toAppRunningOnTestServer'"
 }
 ```

The *npm script command* to run is then
```bash
npm run test:e2e-pipeline:testURL
```

## Conclusions
If using **npx**, beware the differences between **npm** and update the **npx**
command to use the same version the **npm** package uses.

Although **npx** is useful for one-off commands and exploratory stuff, versions
can bite you.

Therefore add a handy generic *npm script command* to package.json that can
be used when trying out new *npm script commands*.