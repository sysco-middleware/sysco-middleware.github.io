---
layout: post
title: Getting started with gRPC. Part 1
categories: gRPC
tags: [Protobuf, gRPC, HTTP/2]
author: PrakharSrivastav
---

# Introduction

gRPC is a RPC platform developed by Google which was made open source (Apache 2.0 license) in early 2015. It consists of two components:- an HTTP/2 based protocol called the gRPC and a data serialization framework called protobuf. By default gRPC uses protobuf for serialization, but it is possible to replace the serialization layer by other content types like Thrift and FlatBuffers.

From gRPC website:

> gRPC is a modern, open source remote procedure call (RPC) framework that can run anywhere. It enables client and server applications to communicate transparently, and makes it easier to build connected systems.




# Repository

Repository contains examples of schema evolution - full compatibility.
[poc-confluent-schema-registry-p1](https://github.com/sysco-middleware/poc-confluent-schema-registry-p1)

## Content: 

1. What is gRPC and why would we use it?  
1.1. Features.  
1.2. Supported programming languages.  
1.3. Supported data formats.
1.4. Typical applications that can be built with gRPC.

2. Fitting pieces together. [General workflow for a simple project]
2.1. Defining message structure.
2.2. Defining service definition.
2.3. Generating client/server stubs in your favourite programming language.
2.4. Implementing client/server logic.

3. Working example. [] 
3.1. Defining message structure (protobuf).  
3.2. Defining service definitions (protobuf).  
3.3. Compiling Grpc client/server stubs out of protobuf.
3.4. More references?

4. Whats is next.
5. Useful references.

## 1. What is gRPC and why would we use it? 

gRPC framework is based on a client-server model of remote procedural calls, where a client can directly call methods on the server application as if it was a local object. The framework inherits transport features from HTTP/2 protocol, utilizing strongly typed schema support provided by protobuf for serialization. 

A gRPC based project usually starts with defining the underlying data-structure and service definition i.e a contract between the client and the server. Next steps would include generating client and server stubs in preferred programming language. Once the client-server stubs are available, developers would concentrate on implementing the contract interfaces (bushiness logic), while ignoring the nuances of infrastructure and transport protocols.

![gRPC client-server communication](/images/2019-07-23-GettingStartedWithProtobufGrpc/gRPC.svg)
[Source](https://grpc.io/)

Apart from providing a robust framework for client-server communication, gRPC provides a tons of other features which make it indispensable choice for modern application development and design. A few of these notable features are detailed below.

### 1.1. Features
Few noticeable features offered by gRPC are:
- Simple and strongly typed message and service definition. Ensures that the contract between client and server remains consistent.
- Api versioning and backward compatibility. If done right, the gRPC apis can be changed transparently without breaking client contracts.
- Supports unary and streaming communication. Also, supports bidirectional streaming.
- Pluggable auth, tracing, load balancing and health checking.
- Works across languages and platforms. Provides code generation in organizations preferred programming language of choice.
- Better performance over the wire as the protocol is inherently binary in nature. It is smaller, faster and more efficient.
- Supports reliable gRPC clients as well as clients for web, android and iOS.
- Cloud native and integrates well wih cloud technologies.

### 1.2. Supported programming languages 
Below are the officially supported programming languages
- C++
- Java (including support for Android) 
- Object-C (for iOS)
- Python
- Ruby
- Go
- C#
- Node.js
- PHP
- Dart (beta)

### 1.3. Supported data formats.  

By default gRPC uses protocol buffers, Google’s mature open source mechanism for serializing structured data. However, since protocol buffers act as serialization layer it is possible to replace it with other content types.

From gRPC website:

> gRPC is designed to be extensible to support multiple content types. The initial release contains support for Protobuf and with external support for other content types such as FlatBuffers and Thrift, at varying levels of maturity.

### 1.4. Typical applications that can be built with gRPC.
The main usage scenario for gRPC are:
- Efficiently connecting polyglot services in microservices style architecture
- Low latency, highly scalable, distributed systems.
- Connecting mobile devices, browser clients to backend services
- Designing a new protocol that needs to be accurate, efficient and language independent.
- Generating efficient client libraries

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

The way is to define schema in file `.avsc`. Definition's syntax is `json`. 

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

## Next posts

* More Complex examples
* Confluent Schema Registry with Kafka Streams API
* REST Proxy Schema registry