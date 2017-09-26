---
layout: post
title: Kafka - Rewind Consumer Offsets
categories: integration
tags: [kafka]
author: jeqo
---

One of the most important features from *Apache Kafka* is how it manages
Multiple Consumers. Each `consumer group` has a current `offset`, that
determine at what point in a `topic` this `consumer group` has consume
messages. So, each `consumer group` can manage its `offset` independently,
by `partition`.

This offers the possibility to rollback in time and reprocess messages from
the beginning of a `topic` and regenerate the current status of the system.    

But how to do it (programmatically)?

<!--more-->

****
Source code: [https://github.com/jeqo/post-kafka-rewind-consumer-offset](https://github.com/jeqo/post-kafka-rewind-consumer-offset)
****

## Basic Concepts

### Topics and Offsets

First thing to understand to achieve Consumer Rewind, is: rewind over what?
Because `topics` are divided into `partitions`. Records sent from `Producers`
are balanced between them, so each partition has its own `offset` index.

Each `record` has its own `offset` that will be used by `consumers` to define
which messages has been consumed from the **log**.

### Consumers and Consumer Groups

Once we understand that `topics` have `partitions` and `offsets` by `partition`
we can now understand how `consumers` and `consumer groups` work.

`Consumers` are grouped by `group.id`. This property identify you as a
`consumer group`, so the `broker` knows which was the last `record` you have
consumed by `offset`, by `partition`.

This partitions allows *parallelism*, because members from a `consumer group`
can consume `records` from `partitions` independently, in parallel.


Before continue, let's check a simple Kafka Producer implemented with Java:

`KafkaSimpleProducer.java`:

```java
public static void main(String[] args) {
    ...
    Properties properties = new Properties();
    properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
    properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

    Producer<String, String> producer = new KafkaProducer<>(properties);

    IntStream.rangeClosed(1, 100)
            .boxed()
            .map(number -> new ProducerRecord<>(
                    "topic-1",
                    number.toString(),
                    number.toString()))
            .map(record -> producer.send(record))
            .forEach(result -> printMetadata(result));
    producer.close();
}
```

This will create 100 `records` in topic `topic-1`, with `offsets` from 0-99

## From Command-Line

In this first scenario, we will see how to manage offsets from *command-line*
so it will give us an idea of how to implement it in our application.

When you're working from the terminal, you can use `kafka-console-consumer` without
`group.id`, a new `group.id` is generated using:
`console-consumer-${new Random().nextInt(100000)}`.
So unless you use the same `group.id` afterwards, it would be as you create a
new consumer group each time.

By default, when you connect to a `topic` as a `consumer`, you
go to the *latest* `offset`, so you won't see any new message until new records
arrive after you connect.

In this case, going back to the beginning of the topic will as easy as add
`--from-beginning` option to the command line:

<script type="text/javascript" src="https://asciinema.org/a/101246.js" id="asciicast-101246" async></script>

But, what happen if you use `group.id` property, it will only work the first time,
but `offset` gets commited to cluster:

<script type="text/javascript" src="https://asciinema.org/a/101248.js" id="asciicast-101248" async></script>

<script type="text/javascript" src="https://asciinema.org/a/101250.js" id="asciicast-101250" async></script>

So, how to go back to the beginning?

We can use `--offset` option to with three alternatives:

```
--offset <String: consume offset>        The offset id to consume from (a non-  
                                           negative number), or 'earliest'      
                                           which means from beginning, or       
                                           'latest' which means from end        
                                           (default: latest)
```

<script type="text/javascript" src="https://asciinema.org/a/101252.js" id="asciicast-101252" async></script>

## From Java Clients

So, from `command-line` is pretty easy to go back in time in the log. But
how to do it from your application?

If you're using Kafka Consumers in your applications, you have to options
(with Java):

* [Kafka Consumer API](http://kafka.apache.org/documentation/#consumerapi)

* [Kafka Streams API](http://kafka.apache.org/documentation/#streamsapi)   

Long story short: If you need stateful and stream processing capabilities,
go with Kafka Streams.
If you need simple one-by-one consumption of messages by topics, go with
Kafka Consumer.

At this moment this are the options to rewind offsets with these APIs:

- Kafka Consumer API support go back to the beginning of the topic, go back
to a specific offset, and go back to a specific offset by timestamps.

- Kafka Streams API only support to go back to the earliest offset of the
`input topics`, and is well explained by [Matthias J. Sax](https://github.com/mjsax)
in his post
[[1]](https://www.confluent.io/blog/data-reprocessing-with-kafka-streams-resetting-a-streams-application/).

So I will focus in options available in `Kafka Consumer`.

A simple Consumer will look something like this:

```java
public static void main(String[] args) {
    Properties props = new Properties();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
    props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

    KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
    consumer.subscribe(Arrays.asList("topic-1"));

    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(100);

        for (ConsumerRecord<String, String> record : records)
            System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
    }
}
```

This will poll by `100ms` for records and print them out.  In this case
it should print 100 records.

Now let's check how to rewind `offsets` in different scenarios. Consumer API has
add `#seek` operations to achieve this behavior. I will show a naive way to use
these operations using flags but it shows the point:

### Rewind to earliest `offset`

The most common options is to go back to the beginning of the topic, that not
always will be `offset=0`. This will depends on the `retention` policy
option that will be clean up old records based on time or size; but
this also deserves its own post.

To go to the beginning we can use `#seekToBeginning(topicPartition)`
operation to go back to earliest offset:

```java
boolean flag = true;

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(100);

    if (flag) {
        consumer.seekToBeginning(
            Stream.of(new TopicPartition("topic-1", 0)).collect(toList()));
        flag = false;
    }

    for (ConsumerRecord<String, String> record : records)
        //Consume record
}
```

Once the seek to beginnning is done, it will reprocess all records from
`topic=topic-1` and `partition=0`.

### Rewind to specific `offset`

If we can recognized the specific `record` (by `partition`)
from where we need to reprocess all the log,
we can use `#seek(topicPartition, offset)` directly.

```java
boolean flag = true;

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(100);

    if(flag) {
        consumer.seek(new TopicPartition("topic-1", 0), 90);
        flag = false;
    }

    for (ConsumerRecord<String, String> record : records)
        System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
}
```

In this case, we will consume from `record` with `offset=90`from
`topic=topic-1` and `partition=0`.

***
NOTE: It could be cumbersome to map all offsets in case that you have
several partitions. Thats why addition of timestamps helps a lot with this.
***

### Rewind to offset by timestamps

What if you don't know exactly the `offset id` to go back to, but you know
you want to go back 1 hour or 10 min?

For these, since release `0.10.1.0`, there are a couple of
improvements [[2]](https://cwiki.apache.org/confluence/display/KAFKA/KIP-32+-+Add+timestamps+to+Kafka+message)
[[3]](https://cwiki.apache.org/confluence/display/KAFKA/KIP-33+-+Add+a+time+based+log+index)
were added and a new operation was added to `Kafka Consumer API`: `#offsetsForTimes`.

Here is how to use it:

```java
boolean flag = true;

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(100);

    if(flag) {
        Map<TopicPartition, Long> query = new HashMap<>();
        query.put(
                new TopicPartition("simple-topic-1", 0),
                Instant.now().minus(10, MINUTES).toEpochMilli());

        Map<TopicPartition, OffsetAndTimestamp> result = consumer.offsetsForTimes(query);

        result.entrySet()
                .stream()
                .forEach(entry -> consumer.seek(entry.getKey(), entry.getValue().offset()));

        flag = false;
    }

    for (ConsumerRecord<String, String> record : records)
        System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
}
```

In this case, we are using a query first to get the offset inside a timestamp (10 minutes ago)
and then using that offset to go back with `#seek` operation.

As you can see, for each operation I have to define the specific `topic partition`
to go back to, so this can get tricky if you have more than one partition, so I
would recommend to use `#offsetsForTimes` in those cases to get an aligned result
and avoid inconsistencies in your consumers.

In the source code, I've added the steps to get partitions by topic that will
help us to reproduce this steps when you have several topics.

****
**References**

1. https://www.confluent.io/blog/data-reprocessing-with-kafka-streams-resetting-a-streams-application/

2. https://cwiki.apache.org/confluence/display/KAFKA/KIP-32+-+Add+timestamps+to+Kafka+message

3. https://cwiki.apache.org/confluence/display/KAFKA/KIP-33+-+Add+a+time+based+log+index

4. https://cwiki.apache.org/confluence/display/KAFKA/FAQ#FAQ-HowcanIrewindtheoffsetintheconsumer

****

***
Published originally here: [https://jeqo.github.io/post/2017-01-31-kafka-rewind-consumers-offset/](https://jeqo.github.io/post/2017-01-31-kafka-rewind-consumers-offset/)

***
