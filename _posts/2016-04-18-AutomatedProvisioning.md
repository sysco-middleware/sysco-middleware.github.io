---
layout: post
title: Benefits of Automated Oracle FMW Provisioning
categories: oracle soa osb
tags: [Oracle, OPatch, OSB, SOA, Fusion Middleware, Vagrant, Puppet]
author: gugalnikov
---

Oracle Fusion Middleware provisiong is always a critical prerequisite which will substantially influence the success or failure of our development projects. Those of us who have spent many years working with this toolset in its many versions, should know for sure what a distressful experience it is to work with sloppily or incorrectly provisioned environments.

Provisioning can also consume a lot of our precious time, whether it is performed locally or in controlled environments belonging to our organization / customer. As the components have evolved, setup options have also become increasingly complex and diverse (although maybe friendlier from a UI perspective), and even though we may have mastered this craft and are capable of producing a nice and shiny configuration, replicating this consistently and for multiple environments where we can expect high variance regarding product versions, particular requirements, limitations and criticality levels, is without any doubt a very challenging and potentially error-prone endeavor. Add dependencies, intangibles and deadlines to the mix and this can become as complicated as any other project task.  

Nevertheless, for the time being and with all the tools at our disposal, this provisioning processes can be easily streamlined and automated, so we can stop the suffering while also learning some really exciting stuff and providing value to our organization / customer.

# Automated provisioning: what are we looking for?

This "value" we've mentioned may represent lots of things when talking about an optimized provisioning cycle, for example:

- Agility / Speed: which will also translate into developer productivity, time to market and enhanced DR / scaling capabilities.
- Consistency / Standardization: so we can focus mostly on resolving business-oriented challenges rather than tripping up with environment-related issues.
- Change management: being able to evolve our environments by patching, upgrading and fine tuning in an orderly fashion, and without the fear of it collapsing like a                       house of cards at the minimum alteration. 
- Competency building: so your team will be able to learn, perform and improve well-delimited and highly repeteable tasks rather than playing "heroball" (where                               everyone and everything ends up depending on a single engineer's prowess and availability, sound familiar?)

# So, which options do we have?

There are so many, but let's talk about some of them and provide some examples and references. For instance, we will always have the good old config wizard:

![](/images/2016-04-18-AutomatedProvisioning/2016-04-18-config.gif)  

Which may be fine in some cases, but depends on a lot of things in order to run smoothly: user permissions, O.S. parameters, JRE/JDK, environment variables, RCU, network performance, etc. Also, we may not always have graphic capabilities available, so at a loss for other options, its possible we may be stuck with the dreaded console mode, good luck with that!

![](/images/2016-04-18-AutomatedProvisioning/2016-04-18-console.gif)

But there are in fact other options, most notably WLST, which has been around for a long time but can now be easily leveraged by combining its product-specific potential with the flow control capabilities of several provisioners the likes of: Puppet, Ansible, Chef, etc.

So a puppet script for example could take care of the following activities in automated fashion:

- O.S. Setup
- Java Installation
- RCU
- FMW Installation (WLS, SOA, OSB, etc...)
- Patching
- Domain creation and configuration
- Domain packing / unpacking
- Node manager configuration / startup
- Security & Logging
- Admin server & Managed server startup
- JDBC Data Source creation
- And much more...

![](/images/2016-04-18-AutomatedProvisioning/2016-04-18-puppet.png)

Specific properties and configuration details can usually be stored in "properties" files (hiera for example in puppet's case), in order to maximize the potential for reuse and to eliminate hardcoding from our scripts.

![](/images/2016-04-18-AutomatedProvisioning/2016-04-18-hiera.png)

Most of these activities could also be run in parallel for multiple nodes (computers) in some scenarios, so depending on topology, processor speed and possibly network, we could be able to have a ready-made, fully operational, pre-patched & certified environment in 20 to 25 minutes. 
     
You could even go further and pair your working scripts with sophisticated virtualization / container technologies such as Vagrant, Docker, etc., so even the creation and management of boxes can be part of the streamlined process. Vagrant for example can work wonders for spinning up sandbox, development or testing environments, where its management capabilities will let us perform activities such as: private network creation, port forwarding, shared storage provisioning and more with only basic knowledge of these technologies.

Here's what a Vagrantfile looks like:

![](/images/2016-04-18-AutomatedProvisioning/2016-04-18-vagrant.png)

A lightweight virtualization manager like Vagrant, will let you choose between a set of virtualization providers and/or provisioners, or even create your own custom plugins if you are up to it.

Another very positive thing about all of this, is that you can benefit from the constant contribution of the community. There are tons of ready made scripts, base boxes, container images, playbooks, etc... out there, as well as tutorials, blogs, documentation and very valuable resources, even official ones from Oracle itself.

![](/images/2016-04-18-AutomatedProvisioning/2016-04-18-oracle.png)

So why not go look at some examples and try them out?; my recommendation is to do it, so here are some useful links to the fantastic work of some of my good friends and fellow Oracle ACEs (including my Sysco teammate Jorge) who have done some really groundbreaking work in this regard:

[Jorge Quilcate](https://jeqo.github.io/blog/)
[Edwin Biemond](http://biemond.blogspot.no)
[Lucas Jellema](https://technology.amis.nl/author/lucas-jellema/)
[Andreas Koop](http://multikoop.blogspot.no)

I certainly hope you found this post interesting, thanks for reading!!
