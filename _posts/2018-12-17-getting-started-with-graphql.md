---
layout: post
title: Getting started with Graphql.
categories: Graphql , java
tags: [Graphql, Java]
author: PrakharSrivastav
---
# Motivation and target audience

This blog post is targeted towards the architects and the back-end developers who want to model a backend API to be consumed by a frontend UI (web, iOS or android). Although, the focus of this blog post would be on creating graphql APIs, there would be few pointers for the frontend developers on how to interact with the graphql powered backends.

The motivation behind writing this blog post is to gently introduce the Graphql concepts to the target audience and discuss advantages and disadvantages of graphql over traditional REST APIs. We will then highlight a typical workflow that can be used to model graphql powered APIs and end the post with a simple example to create graphql APIs using Java. Wherever possible, we will provide adequate references to more detailed resources.

The code discussed in this blog post can be found on [github](https://github.com/PrakharSrivastav/blog-graphql)

# Repository

The examples provided in this blog are based on [this](https://github.com/PrakharSrivastav/blog-graphql) Github repository.

You should use this blog post to grab the Graphql concepts and the Github repository to validate the concepts you have learned in this blog post. The code-base contains the instructions on how to run the server and the client in [README](https://github.com/PrakharSrivastav/blog-graphql/README.md) and provides ample comments wherever possible.

# Introduction

1. Provide simple explanation
2. Provide simple comparison with rest
![graphql-arch](/images/2018-12-17-getting-started-with-graphql/graphql-arch.png)
[Image source](https://blog.apollographql.com/how-do-i-graphql-2fcabfc94a01)
3. Provide more details


![graphql](/images/2018-12-17-getting-started-with-graphql/graphql.png)
[Image source](https://graphql.org/)

From Graphql website:
> GraphQL is a query language for your API, and a server-side runtime for executing queries by using a type system you define for your data. GraphQL isn't tied to any specific database or storage engine and is instead backed by your existing code and data.

## Content:

1. What is gRPC and why would we use it?
1.1. Features.
1.2. Supported programming languages.
1.3. Supported data formats.
1.4. Typical applications that can be built with gRPC.

2. Fitting pieces together. [General workflow for a simple project]
2.1. Defining payload and service definition.
2.2. Generate gRPC code from protobuf definition.
2.3. Implement gRPC server.
2.4. Implement gRPC client.
2.5. Run the server and call it using a client.

3. Where to go next.
4. Sources and References.

## 1. What is gRPC and why would we use it?

If you have worked with REST APIs, what you are generally used to is:
```
REST API = HTTP1.1 + JSON + REST
```
where `HTTP1.1` is the transport protocol, `JSON` is message format, and `REST` is the architectural style (resourceful endpoint).

gRPC handles the same scenario as:
```
gRPC API = HTTP/2 + Protobuf + RPC
```
Here the transport protocol is based on `HTTP/2`, the data format is defined using `Protobuf` and architectural style is `RPC`.

Few advantages of this approach are:
- 1. **Transport level optimization**: `HTTP/2` is more performant that standard `HTTP1.1`. One of the major advantages is that HTTP/2 creates a single connection between client and server and both parties can exchange data over a single connection, whereas with HTTP1.1 each data exchange needs a separate connection. Check the link at end of this section for more detailed explanation.
- 2. **Efficient serialization**: `Protobuf` is a language-neutral way for serializing structured data. They are also very efficient and packed, while JSON, on the other hand, is textual inherently. Protobuf enables us to write simple schema definitions with an added advantage that protobuf compiler can translate your data definitions to the language of your choice. Therefore, you create your structure once and use it across all supported languages.
- 3. **RPC style communication**: `REST` encourages strict CRUD operations on a resource, however with `RPC` server can provide more general computation functions like `generateReport()`.


References:
- [http2-vs-http1](https://imagekit.io/blog/http2-vs-http1-performance/)
- [comparing-serilization-formats](http://labs.criteo.com/2017/05/serialization/)
- [rest-vs-rpc-style](https://apihandyman.io/do-you-really-know-why-you-prefer-rest-over-rpc/)

### 1.1. Features.

Apart from providing a robust framework for client-server communication, gRPC provides a few other features which make it an indispensable choice for modern application development.

- Strongly typed payload and service definition. Ensures consistent contract between the client and server.
- API evolution, versioning, and backward compatibility. gRPC APIs can be changed transparently without breaking client contracts.
- Supports unary and streaming communication with bidirectional streaming support.
- Pluggable auth, tracing, load balancing and health checking.
- Code generation for multiple programming languages.
- More performant than traditional REST+JSON.
- Supports reliable gRPC clients as well as clients for web, Android, and iOS.
- Cloud native and integrates well with cloud technologies.

### 1.2. Supported programming languages.

Below are the officially supported programming languages:
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

Other third-party supported languages:
- Haskell
- Erlang
- Scala
- Elixir
- Rust
- Elm
- TypeScript

### 1.3. Supported data formats.

By default, gRPC uses protocol buffers, Google’s mature open source mechanism for serializing structured data. However, since protocol buffers act as serialization layer, it is possible to replace it with other content types or provide your own implementations.

From gRPC website:

> gRPC is designed to be extensible to support multiple content types. The initial release contains support for Protobuf and with external support for other content types such as FlatBuffers and Thrift, at varying levels of maturity.

References:
- [gRPC-with-flatbuffers](https://grpc.io/blog/flatbuffers)

### 1.4. Typical applications that can be built with gRPC.

The main usage scenario for gRPC are:
- Efficiently connecting polyglot services in microservices style architecture.
- Low latency, highly scalable, distributed systems.
- Connecting mobile devices, browser clients to backend services.
- Designing a new protocol that needs to be accurate, efficient and language independent.
- Generating efficient client libraries.

## 2. Fitting pieces together.

A typical workflow for creating gRPC services is defined below. These steps show how protobuf, gRPC and your favorite programming language fit together.

- Define payload (request/response structure) and service (operations exposed by service) definition in .proto file. This defines a contract between the client and the server.
- Use a compiler to generate gRPC code from .proto files.
- Implement the server in one of the supported languages.
- Implement a client that talks to service through the generated stub.
- Run the server and call it using the client.

Let us discuss these steps in more detail in the next few sections.

### 2.1. Define payload and service definition.

In any typical gRPC project, we start with defining our payload in protobuf format. The payload will define our request and response structures and will be stored in .proto files (For eg invoice.proto ). An important thing to note here is that each field in our payload has a data-type, name, and id. Doing this ensures that our payload is strongly typed and the protocol reports an error during compile time if the client uses the wrong datatype while invoking methods on the server. Protobuf uses the term `message` to define a payload. Below we have defined a message called Invoice with fields id, amount and customerId.
```
message Invoice {
    string id = 1;
    float amount = 2;
    string customerId = 3;
}
```

Next, we define our service and the method it exposes. The arguments and return types will always be a message that we have defined earlier. Below we have defined a service called InvoiceService and defined a method (or operation) called `Pay`. `Pay` method accepts an input of type `Invoice` and has a return type as `Invoice`.
```
service InvoiceService {
    rpc Pay(Invoice) returns (Invoice);
}
```
Once we generate code from protobuf, these strongly typed methods will be available for our client and server for implementation. This will ensure that both the client and the server stubs adhere to the contract defined in protobuf.

Before proceeding further, let us take a quick look at what behaviors can our operations possess:
- **Unary**: This refers to a blocking synchronous call made by the client to the gRPC server. The Pay method above uses `rpc` keyword to determine the Unary behavior for this request. In simplified terms, this means that a client will make a request, and wait until the server responds.
- **Streaming**: This can be achieved in three different flavors. Client pushing messages to stream; Server pushing messages to a stream or bidirectional. In all these cases, the client will initiate the request, and the behavior will be determined by the presence of `stream` keyword in the method definition.

One thing to keep in mind while creating APIs is ensuring their forward and backward compatibilities. It is important that any minor API changed should not break the existing clients. Protobuf provides a few guidelines to seamlessly evolve your APIs. Below mentioned reference will point you in the correct direction if you want to learn more about these concerns:
- [Protobuf API versioning](https://www.beautifulcode.co/backward-and-forward-compatibility-protobuf-versioning-serialization)
- [Google API design guide](https://cloud.google.com/apis/design/)
- [Evolving Protobuf APIs](https://blog.envoyproxy.io/evolving-a-protocol-buffer-canonical-api-e1b2c2ca0dec)

References below will provide more details on how to use protocol buffers to structure data.
- [Protocol buffer language guide](https://developers.google.com/protocol-buffers/docs/proto3)
- [Data types](https://developers.google.com/protocol-buffers/docs/proto#scalar)
- [Style guide](https://developers.google.com/protocol-buffers/docs/style)

### 2.2. Generate gRPC code from protobuf.

Now that we have written our payload and service definition, its time to generate code in our preferred programming language from protobuf. We will use Java as a programming language throughout this blog, but the process to generate code in other languages would just be just setting a flag away.

There are 2 major ways to generate code from protobuf:
- Using **protoc compiler**: Install protoc compiler *(download link provided below). To generate code using the compiler, we need to provide the path to the source, destination, and .proto files. To generate java classes from protobuf we will issue a command like `protoc -I=$SRC_DIR --java_out=$DST_DIR $SRC_DIR/invoice.proto`. This command will read the protobuf definition under `$SRC_DIR/invoice.proto` and generate java classes under `$DST_DIR`. To generate code in other languages we replace the --java_out flag with --go_out or --js_out flag.
- Using **build tools** like Gradle, sbt or maven. These tools provide tasks to directly generate source code in your preferred programming language.

A good practice is to install protoc compiler and create shell/make scripts to generate the source code in your preferred language. We will cover that approach in one of the upcoming blogs in detail. In this blog, we will generate java code using gradle build system.

References:
- [protoc compiler download](https://developers.google.com/protocol-buffers/docs/downloads)
- [protoc compiler reference](https://developers.google.com/protocol-buffers/docs/javatutorial)
- [protobuf-gradle-plugin](https://github.com/google/protobuf-gradle-plugin)

### 2.3. Implementing gRPC Server.

The code generated in the previous section will have two java files, `InvoiceServiceGrpc.java` and `InvoiceOuterClass.java`. The former will contain the service definition, while the latter will have payload definition. Implementing the gRPC server will consist of 2 basic steps:

- **Implementing the contract**. This means overriding the service base class generated from our service definition and doing the actual "work" of our service by providing business logic.
- **Bootstrapping the server**. This means adding the implementation done in the previous step as service and running a gRPC server to listen for requests from clients and return the service responses.

Below snippets show a quick and dirty example of implementing gRPC server.
```
/**Implementing contract**/
public class InvoiceServiceImpl extends InvoiceServiceGrpc.InvoiceServiceImplBase {
    public InvoiceServiceImpl() {}
    @Override
    public void pay(InvoiceOuterClass.Invoice request, StreamObserver<InvoiceOuterClass.Invoice> responseObserver) {
        // request argument represents typed client request
        // responseObserver is a means to control streaming behavior in gRPC

        // Use a builder to construct a new Protobuf object. Here we provide a custom business logic to increase amount by 1
        InvoiceOuterClass.Invoice response = InvoiceOuterClass.Invoice.newBuilder()
                .setAmount(request.getAmount() + 1)
                .setCustomerId(request.getCustomerId())
                .setId(request.getId())
                .build();

        // Use responseObserver to send a single response back
        responseObserver.onNext(response);

        // When you are done, you must call onCompleted.
        responseObserver.onCompleted();
    }
}
```
```
/**Bootstrapping the server and added the service implementation**/
public class Server {
    public static void main(String[] args) throws IOException, InterruptedException {
        io.grpc.Server server = ServerBuilder.forPort(8080)     // listen on port 8080
                .addService(new InvoiceServiceImpl())           // add service implementation
                .build();
        server.start();                                         // start server
        server.awaitTermination();                             
    }
}
```

### 2.4. Implementing gRPC Client.

Implementing the gRPC client consists of 2 steps.
- **Create gRPC channel**: This is to define the server address and the port we want to connect to.
- **Creating stubs**: Use the channel to create stubs to communicate with the server. The stubs can again be of 2 types depending on whether the operation is defined as *Unary* or *Streaming* type.
    - a *blocking/synchronous* stub: this means that the RPC call waits for the server to respond, and will either return a response or raise an exception.
    - a *non-blocking/asynchronous* stub that makes non-blocking calls to the server, where the response is returned asynchronously. You can make certain types of streaming call only using the asynchronous stub.

Let's have a look at the below snippet to understand the anatomy of the gRPC client.
```
/**Implementing a gRPC client**/
public class Client {
    public static void main(String[] args) throws InterruptedException {
        // 1. Creating a gRPC channel
        final ManagedChannel channel = ManagedChannelBuilder
                .forAddress("localhost", 8080)      // Set host and port
                .usePlaintext()                     // This setting should be used in dev. In prod, this should be replaced with TLS/Certificate
                .build();

        // 2. Create a synchronous stub
        final InvoiceServiceGrpc.InvoiceServiceBlockingStub stub = InvoiceServiceGrpc.newBlockingStub(channel);

        // 3. Prepare request
        final InvoiceOuterClass.Invoice request = InvoiceOuterClass.Invoice.newBuilder()
                .setAmount(100.00f)
                .setId("123")
                .setCustomerId("XYZ123")
                .build();

        // 4. Call the pay method on the stub
        final InvoiceOuterClass.Invoice response = stub.pay(request);
        System.out.println(response);

        // Finally close the channel
        channel.shutdown();
    }
}
```
### 2.5. Run the server and call it using client.  

Now that we have completed the implementation for our client and server, its time to run them and check out results. For the purpose of this blog, we will run the client and server from our IDE and have a look at the output produced by our server.

Start the gRPC server by running the main method in Server.java
![Start Server](/images/2018-07-27-GettingStartedWithProtobufGrpc/Server.png)

Now run the client by running the main method in Client.java
![Run Client](/images/2018-07-27-GettingStartedWithProtobufGrpc/Client.png)

You can notice the difference in the value of the amount in the response. On the server, our business logic increments the amount by 1

Let's check server logs that were generated while processing this request
![Server Logs](/images/2018-07-27-GettingStartedWithProtobufGrpc/ServerResponse.png)

## 3. Where to go next?

The example presented in this blog is extremely basic and was intended to ease the learning curve for the beginners. It is focussed towards acquainting the readers with basic terminologies in gRPC and presenting a basic workflow to play around and get started with.

At this point we recommend you to clone the repository at [Github](https://github.com/sysco-middleware/workshop-grpc) and work through the instructions provided in README.md file. It prototypes a more advanced Invoice Generation system with step by step explanation on data-structure modeling for gRPC, setting up a java project for gRPC using Gradle and unit testing your gRPC code.

Now that we have a basic understanding of what gRPC is. We will provide more blog posts and examples covering different aspects like message modeling, validation, and error handling in gRPC,  protoc compilation techniques, performance optimization etc.

## 4. Sources and References.

- Grpc website : [grpc](https://grpc.io/)
- Google protobuf documentation : [google-protocol-buffers](https://developers.google.com/protocol-buffers/)
- Curated list of gRPC tools and plugins : [grpc-ecosystem](https://github.com/grpc-ecosystem/awesome-grpc)

- This blog post on gRPC: [what-is-grpc](https://rms1000watt.github.io/post/what-is-grpc/)
- These interesting stackoverflow questions : [how-is-grpc-different-from-rest](https://stackoverflow.com/questions/43682366/how-is-grpc-different-from-rest) , [grpc-and-zeromq-comparsion](https://stackoverflow.com/questions/39350681/grpc-and-zeromq-comparsion), [is-grpchttp-2-faster-than-rest-with-http-2](https://stackoverflow.com/questions/44877606/is-grpchttp-2-faster-than-rest-with-http-2)
