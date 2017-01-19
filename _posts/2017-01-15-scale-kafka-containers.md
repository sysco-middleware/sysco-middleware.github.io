---
layout: post
title: Scaling Kafka with Docker Containers
categories: devops integration
tags: [kafka, docker]
author: jeqo
---

In this post I will show how to use Docker containers to create and scale
a Kafka cluster, and also how to create, scale and move `topics` inside
the cluster.

<!--more-->

***
Repository: https://github.com/jeqo/post-scale-kafka-containers

***

# Single-Node Cluster

First of all, let's start with the most simple way to run Docker, that
could be useful for some development scenarios: **Single-Node Cluster**

Apache Kafka architecture is based in 2 main components: The *Apache
Kafka server* itself, and the *Apache Zookeeper server* used for internal
coordination.

That's why a Kafka single-node cluster requires at least a
couple of processes.

If we talk in Container terms and practices, these processes should be
run in 2 different containers.

The easiest way to do this is defining these processes as
Docker Compose services is a `kafka-cluster/docker-compose.yml` file:

***
I will use a couple of Docker images. These are fairly simple
and you can find their source code here:
[Apache Kafka](https://github.com/jeqo/docker-image-apache-kafka),
[Apache Zookeeper](https://github.com/jeqo/docker-image-apache-zookeeper), and
[Confluent Platform](https://github.com/jeqo/docker-image-confluent-platform)
***

```yaml
version: "2.1"
services:
  kafka:
    image: jeqo/apache-kafka:0.10.1.0-2.11
    links:
      - zookeeper
  zookeeper:
    image: jeqo/apache-zookeeper:3.4.8
```

This configuration defines 2 services: `kafka` and `zookeeper`. The `kafka`
service `link` and environment variable `ZOOKEEPER_CONNECT` configure the access
from `kafka` to `zookeeper` service.

If we try to start these configuration with `docker-compose up -d`,
Docker Compose will create a `network` where these service can communicate.

```bash
jeqo@jeqo-Oryx-Pro:.../single-node-kafka-cluster$ docker-compose up -d
Creating network "kafkacluster_default" with the default driver
Creating kafkacluster_zookeeper_1
Creating kafkacluster_kafka_1
```

If you want to communicate with the cluster from your application's
docker-compose configuration, you can do it as follows:

```yaml
version: "2.1"
services:
  kafka:
    image: jeqo/apache-kafka-client:0.10.1.0-2.11
    command: sleep infinity
    networks:
      - default
      - kafkacluster_default #(2)
networks: #(1)
  kafkacluster_default:
    external: true
```

Here we define first an `external network` called `singlenodekafkacluster_default`
that will give us access to the kafka cluster network. Then we add this network
to the service network.

To test our client, start it up running `docker-compose up -d` and then connect
to the cluster with the following command:

```bash
$ docker-compose exec kafka bash
# bin/kafka-console-producer.sh --broker-list kafka:9092 --topic topic1
test
# bin/kafka-topics.sh --zookeeper zookeeper:2181 --list      
topic1
```

# Multi-Node Cluster

To scale a container using Docker Compose is as simple as using the `scale` command:

```bash
docker-compose scale kafka=3
```

This will create 2 more containers:

```bash
$ docker-compose scale kafka=3
Creating and starting kafkacluster_kafka_2 ... done
Creating and starting kafkacluster_kafka_3 ... done
```

You, as an application developer, only need to know one of the `broker` IPs, or use the service
name to connect to the cluster. As the documentation specifies, the client (eg. producer or consumer)
will use it only once to get the Kafka `broker` IPs from the same cluster. This means that
Kafka scaling will be transparent to your application.

To validate that all brokers are part of the cluster let's use Zookeeper client to check. From
client container:

```bash
$ docker-compose exec kafka bash
# bin/zookeeper-shell.sh zookeeper:2181
ls /brokers/ids
[1003, 1002, 1001]
```

# Scaling Topics

In Kafka, `Topics` are distributed in `Partitions`. `Partitions` allows **scalability**, enabling `Topics`
to fit in several nodes, and **parallelism**, allowing different instances from the same `Consumer Group` to
consume messages in parallel.

Apart from this, Kafka manage how this `Partitions` are replicated, to achieve high availability. In
this case, if you have many `replicas` from one `partition`, one will be the `leader` and there will
be zero o more `followers` spread on different nodes.

How do we configure this using this simple container configuration? Let's evaluate some scenarios:

## Adding new topics to the cluster

Once you scale your cluster, Kafka won't use these new nodes unless new `topics` are created.

Let's test it following these steps:

1. Start a single node cluster

<script type="text/javascript" src="https://asciinema.org/a/9xzqgicktaqhzp1fofjk9ejgm.js" id="asciicast-9xzqgicktaqhzp1fofjk9ejgm" async></script>

2. Then start client, create a topic `topic1`, and describe the topic to check the broker

<script type="text/javascript" src="https://asciinema.org/a/2schnuetb24mjx6txopew51hc.js" id="asciicast-2schnuetb24mjx6txopew51hc" async></script>

3. Scale your cluster to 3 nodes

<script type="text/javascript" src="https://asciinema.org/a/ahibdzz7xt67q53sc5ert6qdp.js" id="asciicast-ahibdzz7xt67q53sc5ert6qdp" async></script>

4. Add topics to occupy other brokers

Using multiple partitions:

<script type="text/javascript" src="https://asciinema.org/a/enq2czkpgdf0tbf3u6fwir3ml.js" id="asciicast-enq2czkpgdf0tbf3u6fwir3ml" async></script>

Or using a replication factor:

<script type="text/javascript" src="https://asciinema.org/a/f0u67h5ufiz4zkup84a1t8t5g.js" id="asciicast-f0u67h5ufiz4zkup84a1t8t5g" async></script>

To decide what `replication factor` or how many `partitions` to use, depends
on your use case. This deserves its own blog post.

## Expanding topics in your cluster

Expanding topics in your cluster means move `topics` and `partitions` once
you have more `brokers` in your `cluster`, because, as show before,
your new `brokers` won't store any data, once they are created, unless
you create new `topics`.

You can do this 3 steps:

1. Identify which topics do you want to move.

2. Generate a candidate reassignment. This could be done automatically, or
you can decide how to redistribute your topics.

3. Execute your reassignment plan.

You can do this following the documentation here: http://kafka.apache.org/documentation/#basic_ops_cluster_expansion

The steps described in the documentation are automated a bit with Ansible:

Inside the `playbooks/prepare-reassignment.yml` file you have 2 variables:

```yaml
vars:
  topics:
    - topic1
  broker_list: 1003
```

This will prepare a recipe to move your topic `topic1` to `broker` with id `1003`.

<script type="text/javascript" src="https://asciinema.org/a/c6332x8t7yumpj65ie4qudgem.js" id="asciicast-c6332x8t7yumpj65ie4qudgem" async></script>

You can paste the JSON file generated into `playbooks/reassign-topic-plan.json`

```json
{
  "version":1,
  "partitions":[{"topic":"topic1","partition":0,"replicas":[1003]}]
}
```

And then run this plan with the another playbook: `playbooks/execute-reassignment.yml`

<script type="text/javascript" src="https://asciinema.org/a/99308.js" id="asciicast-99308" async></script>

# Confluent Platform images

All these could be done in the same way with [Confluent Platform](https://www.confluent.io/).

There is a couple of directories `confluent-cluster` and `confluent-client` to test this out:

<script type="text/javascript" src="https://asciinema.org/a/a446bixdfn3l8xqoiolmsmlqg.js" id="asciicast-a446bixdfn3l8xqoiolmsmlqg" async></script>

Hope this post help you to understand Kafka `topics` and how `containers` can
help you to run clusters in seconds :)

And, you know, run ...

<blockquote class="twitter-tweet" data-lang="es"><p lang="en" dir="ltr">.<a href="https://twitter.com/apachekafka">@apachekafka</a> everywhere :) <a href="https://t.co/AcEmkRBCpv">pic.twitter.com/AcEmkRBCpv</a></p>&mdash; Gwen (Chen) Shapira (@gwenshap) <a href="https://twitter.com/gwenshap/status/777660752626851840">19 de septiembre de 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

***
Published originally here: https://jeqo.github.io/post/2017-01-15-scale-kafka-containers/

***
