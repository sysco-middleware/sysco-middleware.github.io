---
layout: post
title: Getting started with gRPC. Part 1
categories: gRPC
tags: [Protobuf, gRPC, HTTP/2]
author: PrakharSrivastav
---

# Introduction

gRPC is an RPC platform developed by Google which was made open source (Apache 2.0 license) in early 2015. It consists of two components:
- an HTTP/2 based protocol called the gRPC and 
- a data serialization framework called protocol buffers (or protobuf).

By default gRPC uses protobuf for serialization, but it is possible to replace the serialization layer.

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
2.3. Generating client/server stubs in your favorite programming language.
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

A gRPC based project usually starts with defining the underlying data-structure and service definition i.e a contract between the client and the server. Next steps would include generating client and server stubs in the preferred programming language. Once the client-server stubs are available, developers would concentrate on implementing the contracts (business logic), while avoiding the nuances of infrastructure and transport protocols.

![gRPC client-server communication](/images/2019-07-23-GettingStartedWithProtobufGrpc/gRPC.svg)
[Source](https://grpc.io/)

Apart from providing a robust framework for client-server communication, gRPC provides tons of other features which make it an indispensable choice for modern application development and design. A few of these notable features are detailed below.

### 1.1. Features
Few noticeable features offered by gRPC are:
- Simple and strongly typed message and service definition. Ensures that the contract between the client and server remains consistent.
- API versioning and backward compatibility. If done right, the gRPC APIs can be changed transparently without breaking client contracts.
- Supports unary and streaming communication. Also, supports bidirectional streaming.
- Pluggable auth, tracing, load balancing and health checking.
- Works across languages and platforms. Provides code generation in organizations preferred programming language of choice.
- Better performance over the wire as the protocol is inherently binary in nature. It is smaller, faster and more efficient.
- Supports reliable gRPC clients as well as clients for web, Android, and iOS.
- Cloud native and integrates well with cloud technologies.

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

By default, gRPC uses protocol buffers, Google’s mature open source mechanism for serializing structured data. However, since protocol buffers act as serialization layer it is possible to replace it with other content types or provide your own implementations.

From gRPC website:

> gRPC is designed to be extensible to support multiple content types. The initial release contains support for Protobuf and with external support for other content types such as FlatBuffers and Thrift, at varying levels of maturity.

### 1.4. Typical applications that can be built with gRPC.
The main usage scenario for gRPC are:
- Efficiently connecting polyglot services in microservices style architecture
- Low latency, highly scalable, distributed systems.
- Connecting mobile devices, browser clients to backend services
- Designing a new protocol that needs to be accurate, efficient and language independent.
- Generating efficient client libraries

## 2. Fitting pieces together. 

The best way to understand how protocol buffers, gRPC, and your favorite programming language fit together would be to break down the development workflow in simpler steps. The process defined here would be sufficient for most of the small to medium scale projects. The larger projects can utilize teams' experience, expertise, and resources to achieve more fine-grained control and automation.

Main steps involved in development workflow would involve:


### 2.1. Defining message structure.

We start by defining the message structure for our domain models in protobuf format. At this point, we also define the message structure of the requests and the responses for various operations (methods) that our API will support. These definitions are stored in .proto files and will be compiled by gRPC compile to create client-server stubs in our favorite programming language. For eg:

```
message Invoice {
    string id = 1;
    float amount = 2;
    string customerId = 3;
}
```
Here, we have defined Invoice model with 3 fields id, amount and customerId. We have also provided types for each field. It is considered a good practice to strongly type the domain entities and request and response structure. Few considerations to be kept in mind while generating message structure are:
- Define strongly typed domain models. Each field for a domain model should be defined with a name and datatype.
- Extract the constants into Enums. Provide scopes to the Enums.
- Split the domain models into multiple files, packages for better code organization.
- Define strongly typed request and response structures.
- Use protocol buffer syntax 3 as it is most recent and maintained version.

Below resources provide a deep insight on protocol buffer format. We shall see a more relevant example to define a message in section 3.1.
- [Protocol buffer language guide (syntax 3)](https://developers.google.com/protocol-buffers/docs/proto3)
- [Data types](https://developers.google.com/protocol-buffers/docs/proto#scalar)
- [Style guide](https://developers.google.com/protocol-buffers/docs/style)
- [Api Versioning](https://www.beautifulcode.co/backward-and-forward-compatibility-protobuf-versioning-serialization)
- [Few caveats](http://www.golangdevops.com/2017/08/16/why-not-to-use-protos-in-code/)

### 2.2. Defining service definition.  
Next step in our workflow is to define the methods (operations) that our API provides. The arguments to these methods will either be domain entities (created in the previous step), custom request/response structures or simply scalar values. For example, for Invoice domain model created above, we create an InvoiceService and provide RPC method definition.
```
service InvoiceService {
    rpc Pay(Invoice) returns (bool);
}
```
Here, we have defined a method Pay in the InvoiceService. This method takes input as Invoice type and provides a boolean response. As mentioned earlier, we can also create a custom type (like Invoice) for our request and response.

Once we compile our protobuf in next steps, these operations will be available in the client and the server stubs with typed arguments and return types. This will ensure that both the client and the server stubs adhere to the contract defined in protobuf.

Before proceeding further, let us take a quick look at what behaviors can our operations possess:
- **Unary** : This refers to a blocking synchronous call made by the client to the gRPC server. The Pay example above uses `rpc` keyword to determine the Unary behaviour for this request.
- **Streaming** : This can be achieved in three different flavors. Client pushing messages to stream; Server pushing messages to a stream or bidirectional. In all these cases, the client will initiate the request, and the behavior will be determined by the presence of `stream` keyword in the method definition.


More references:
- [gRPC Concepts](https://grpc.io/docs/guides/concepts.html)
- [gRPC Transport Anatomy](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md)

### 2.3. Generating client/server stubs in your favorite programming language.


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