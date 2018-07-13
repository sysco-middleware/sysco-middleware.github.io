---
layout: post
title: Kafka - Confluent Schema Registry. Part 1
categories: Kafka
tags: [Kafka, Schema registry, Apache Avro, Confluent]
author: NikitaZhevnitskiy
---

# Introduction

Let's talk about Kafka Schema Registry essentials.

# Repository

Repository contains examples of schema evolution - full compatibility.
[Kafka-schema-registry-poc](https://github.com/sysco-middleware/kafka-schema-registry-poc)

## Content: 

1. What is schema registry and why we need it?  
1.1. Use case.  
1.2. Full compatibility.  
1.3. Avro serializer.
2. How to define schema?  
2.1. In source code.  
2.2. Using reflection.  
2.3. Using json.
3. Schema evolution example.  
3.1. Up kafka.  
3.2. Up project.
4. Important notes.

## 1. What is schema registry and why we need it?

Kafka schema registry is service(app) which stores schemas and provides RESTful interface to preform CRUD operations. 
Schemas can be applied to key/value or both.
Kafka schema registry is separate node. Kafka Consumer/Producer should have `schema.registry.url` in properties, 
if schema registry is in use.
```
  Properties properties = new Properties();
  properties.setProperty("schema.registry.url", "http://localhost:8081");
```

Kafka schema registry should be fault tolerant.
Why? Short answer is to `centralize` message verification.
Kafka itself is not responsible for data verification, it stores only bytes and publish them.
![Confluent Schema registry](https://cdn-images-1.medium.com/max/1500/1*ZDWhGdCZ-4ZVKxYONcuj0Q.png)
[Source](https://medium.com/@stephane.maarek/introduction-to-schemas-in-apache-kafka-with-the-confluent-schema-registry-3bf55e401321)
### 1.1. Use case
All of us faced with continuously coming new requirements from customers. 
Sometimes we need to change data transfer objects: add additional fields or remove some of them,  
which will cause mapping. 
Schema registry is answer to - how to support schema versions and achieve full compatibility. 
Do all those changes without breaking any dependent parts.

### 1.2. Full compatibility

Full compatibility means that message which is produced with Old-schema can be consumed with New-Schema 
and opposite, message which is produced with New-schema can be consumed using Old-Schema.   

![Full compatibility](/images/2018-07-13-KafkaSchemaRegistryPart1/full_compatibility.png)  

 Full compatibility is a goal in most of cases.

### 1.3. Avro serializer.  

[Apache Avro](https://github.com/apache/avro) is data serializer. This serializer is often used with
Confluent Schema Registry for Kafka. 

Avro key-features:
* Compressed data
* [Types support](https://avro.apache.org/docs/1.8.2/spec.html#schemas)
* Embedded documentation support
* Defined using json 
* Avro object contains schema and data
* Shema evolution 

## 2. How to define schema?

There are 3 ways how to define avro schema.

### 2.1. In source code.   

```
Schema.Parser parser = new Schema.Parser();
Schema schema = parser.parse("{\n"
  + "   \"type\": \"record\",\n"
  + "   \"namespace\": \"no.sysco.middleware.poc.kafkaschemaregistry.avro\",\n"
  + "   \"name\": \"Business\",\n"
  + "   \"version\": \"1\",\n"
  + "   \"doc\":\"Business record contains name of company and list of customers\",\n"
  + "   \"fields\": [\n"
  + "     { \"name\": \"company_name\", \"type\": \"string\", \"doc\": \"Name of company\" },\n"
  + "     {\n"
  + "        \"name\": \"customers\",\n"
  + "        \"doc\": \"List of customers\",\n"
  + "        \"type\": {\n"
  + "          \"type\": \"array\",\n"
  + "          \"items\": {\n"
  + "            \"type\": \"record\",\n"
  + "            \"name\":\"Customer\",\n"
  + "            \"fields\":[\n"
  + "              { \"name\": \"first_name\", \"type\": \"string\", \"doc\":\"Customer name\" },\n"
  + "              { \"name\": \"last_name\", \"type\": \"string\", \"doc\": \"Customer last name\" }\n"
  + "            ]\n"
  + "          }\n"
  + "        }\n"
  + "     }\n"
  + "   ]\n"
  + "}");
```

[Full example](https://github.com/simplesteph/kafka-avro-course-udemy/blob/master/avro-examples/src/main/java/com/github/simplesteph/avro/generic/GenericRecordExamples.java)

### 2.2. Using reflection.  

Use reflection. Make schema from POJO class. Do not forget to create empty constructor.

```
Schema schema = ReflectData.get().getSchema(ReflectedCustomer.class);
```

[Pojo class example](https://github.com/simplesteph/kafka-avro-course-udemy/blob/master/avro-examples/src/main/java/com/github/simplesteph/avro/reflection/ReflectedCustomer.java)  
[Reflection full example](https://github.com/simplesteph/kafka-avro-course-udemy/blob/master/avro-examples/src/main/java/com/github/simplesteph/avro/reflection/ReflectionExamples.java)

### 2.3. Using json.

Common practise is to define schema in file `.avsc`. Definition's syntax is `json`. 

[Avro-maven-plugin](https://mvnrepository.com/artifact/org.apache.avro/avro-maven-plugin) generates classes from `.avsc`. 

```
{
   "type": "record",
   "namespace": "no.sysco.middleware.poc.kafkaschemaregistry.avro",
   "name": "Business",
   "version": "1",
   "doc":"Business record contains name of company and list of customers",
   "fields": [
     { "name": "company_name", "type": "string", "doc": "Name of company" },
     {
        "name": "customers",
        "doc": "List of customers",
        "type": {
          "type": "array",
          "items": {
            "type": "record",
            "name":"Customer",
            "fields":[
              { "name": "first_name", "type": "string", "doc":"Customer name" },
              { "name": "last_name", "type": "string", "doc": "Customer last name" }
            ]
          }
        }
     }
   ]
}
```

## 3. Schema evolution.

### Up kafka

Kafka should be `UP`, before perform next steps. There are two options how to do it:  
1. `Landoop` - kafka cluster using ladoop images: with UI on [http://localhost:3030](http://localhost:3030), Connectors and additional tools.  
`docker-compose -f docker-compose.landoop.yml up -d`  
2. `Confluent` - kafka cluster using confluent images  
`docker-compose -f docker-compose.confluent.yml up -d`  

### Evolution 

Build project and generate classes from [business-v1.avsc](https://github.com/sysco-middleware/kafka-schema-registry-poc/blob/master/version1/src/main/resources/avro/business-v1.avsc) and [business-v2.avsc](https://github.com/sysco-middleware/kafka-schema-registry-poc/blob/master/version2/src/main/resources/avro/business-v2.avsc). 

`./mvnw clean package`  

Run producers in modules `version1` and `version2` in any sequential order to produce messages to one topic. 
Run consumers in modules `version1` and `version2` in any sequential order to see consumption results.

## 4. Important notes.

These important notes help to achieve full compatibility
1. Make your primary key required
2. Give default values to all the fields that could be removed in the future
3. Be very careful when using ENUM as the can not evolve over time
4. Do not rename fields. You can add aliases instead
5. When evolving schema, ALWAYS give default values
6. When evolving schema, NEVER remove, rename of the required field or change the type

## Refs & sources

* [Schema Registry doc](https://docs.confluent.io/current/schema-registry/docs/index.html)
* [Apache Avro doc 1.8.2](https://avro.apache.org/docs/1.8.2/index.html)
* [Stephane Maarek](https://medium.com/@stephane.maarek/introduction-to-schemas-in-apache-kafka-with-the-confluent-schema-registry-3bf55e401321)

## Next chapters

* More Complex examples
* Kafka Schema Registry with Streams API
* REST Proxy Schema registry