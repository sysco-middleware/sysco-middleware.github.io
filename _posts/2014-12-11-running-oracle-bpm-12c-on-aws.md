---
layout: post
title: Running Oracle BPM 12c on AWS EC2
categories: devops
tags: [chef, vagrant, oracle, bpm, 12c, aws, ec2]
author: jeqo
keep: no
---

In this post, I will show how to create an AWS EC2 Instance with an Oracle BPM 12c Quickstart Domain created. And I will use previous post for related tasks.

Lets see how to achieve this and make this process reusable. These are the steps:

* Create an AWS EC2 instance (with Vagrant)
* Connect to an NFS instance to get the installer (with Chef)
* Install Oracle BPM 12c Quickstart and create a Domain (with Chef)

[Read more...](http://jeqo.github.io/blog/cloud/run-bpm-12c-aws/)
