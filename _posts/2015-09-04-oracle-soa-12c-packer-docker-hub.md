---
layout: post
title: Oracle SOA Suite 12c Docker Image built with Packer
categories: devops
tags: [oracle, soa, docker, packer]
---

After find some limitations on building SOA Docker image using Dockerfile
(as volume access, default size image) I researched on how to improve
building process and I found [Packer](https://packer.io/)
(from the same guy that creates Vagrant).

To check why using Packer instead of Dockerfiles, [go here](http://mmckeen.net/blog/2013/12/27/advanced-docker-provisioning-with-packer/).

I also moved [OracleSOA directory](https://github.com/jeqo/oracle-docker/tree/master/OracleSOA)
from my forked [oracle-docker repository](https://github.com/oracle/docker) to
an independent repository: [github.com/jeqo/oracle-soa-12c-docker](https://github.com/jeqo/oracle-soa-12c-docker).

## Improvements

Basically Dockerfile scripts were moved to shell scripts and are called
from Packer.

```json
  "provisioners": [
    {
      "type": "shell",
      "scripts": [
        "scripts/create-user.sh"
      ]
    },
    {
      "type": "file",
      "source": "./files/",
      "destination": "/u01/"
    },
    {
      "type": "shell",
      "scripts": [
        "scripts/install-java.sh"
      ],
      "environment_vars": [
        "JAVA_RPM=/data/{{user `java_rpm`}}"
      ]
    },
    {
      "type": "shell",
      "scripts": [
        "scripts/install-soa.sh"
      ],
      "environment_vars": [
        "SOA_ZIP=/data/{{user `soa_zip`}}",
        "SOA_PKG={{user `soa_pkg`}}",
        "SOA_PKG2={{user `soa_pkg2`}}",
        "JAVA_HOME=/usr/java/default",
        "MW_HOME=/u01/oracle/soa"
      ]
    }
  ]
```

After provisioning, you can save your image, and if
you want push it to Docker Hub:

```json
  "post-processors": [
    [
      {
        "type": "docker-tag",
        "repository": "jeqo/oracle-soa-12c",
        "tag": "12.1.3-dev"
      },
      "docker-push"
    ]
  ]
```

Very simple and straightforward JSON configuration.

Now you have a Docker image with SOA installer, ready
to create your domain as [this sample](https://github.com/jeqo/oracle-soa-12c-docker/tree/master/samples/12c-domain) explained
on [my previous post](http://jeqo.github.io/blog/devops/docker-image-oracle-soa/).


Source: [jeqo.github.io/blog](http://jeqo.github.io/blog)
