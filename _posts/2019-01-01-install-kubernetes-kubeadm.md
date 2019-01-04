---
layout: post
title: Installing on-premise Kubernetes cluster using kubeadm
categories: kubernetes, containers, orchestration
tags: [kubernetes, containers, orchestration]
author: PrakharSrivastav
---
# Motivation and target audience

This blog post is geared towards developers and administrators who want to setup an on-premise Kubernetes cluster. This post will guide you towards setting up a multi-node kubernetes cluster on linux servers and will address some of the key concerns during the setup.

Since Kubernetes is a huge topic in itself and therefore this post will not cover all the nitty-gritty details in setting up the kuberneters clusters. This blog post is aimed to provide a convenient starting point. The installation steps discussed in this example can be found on [github](https://github.com/PrakharSrivastav/kubernetes-setup).


# Repository

The installation steps provided in this blog are based on [this](https://github.com/PrakharSrivastav/kubernetes-setup) Github repository.

# Introduction

Kubernetes (K8s) is a container orchestration platform that helps users to build, deploy, scale and manage containerized applications and their dynamic life cycle. It was first developed at Google, but later on offered as a seed technology to Cloud Native Computing Foundation (CNCF). 

From Kubernetes  website:

> Kubernetes (k8s) is an open-source system for automating deployment, scaling, and management of containerized applications. It groups containers that make up an application into logical units for easy management and discovery. Kubernetes builds upon 15 years of experience of running production workloads at Google, combined with best-of-breed ideas and practices from the community.

Some of the distinguished kubernetes features are:
- **Service discovery and load balancing**. 
- **Automatic binpacking**.
- **Storage orchestration**. 
- **Self-healing**. 
- **Automated rollouts and rollbacks**.
- **Secret and configuration management**.
- **Batch execution**.
- **Horizontal scaling**.

You can read more about Kubernetes features [here](https://kubernetes.io/). If you are interested in some case studies you can find them [here](https://kubernetes.io/case-studies/). In the next section we will start with installing the kubernetes cluster on linux.

## Content:

- Prerequisites.
    - Verify hardware
    - Configure hostname
    - Configure host files
    - Verify MAC and product_uuid
    - Disable SELinux and Swap
- Install kubernetes and other dependencies.
- Configure kubernetes master.
- Initialize cluster.
- Add nodes to the cluster.
- Create and test pods.
- Where to go next.
- Sources and References.

## 1. Prerequisites

In this example we will set up a kubernetes cluster with one master and 2 nodes. We will use the below configurations to setup our cluster

### Verify server hardware
We will use 3 CentOS 7 servers with minimum 2 CPU and 2 GB RAM. You should have root privileges on the servers to install the required software packages.

For this example we have provisioned 3 server with below IPs
- 192.168.37.48
- 192.168.37.49
- 192.168.37.50 

In the next step we will provide them a unique hostname.

**Note** : The hardware mentioned is to setup basic minimum configuration for kubernetes. Kubernetes comes with lots of bells and whistles and if you are installing all bells and whistles, please refer to this documentation for more details.

### Configure hostname on each of the server
Login to each of the server and change the hostname by following below 2 steps. For example on server 192.168.37.48 we have setup hostname kubernetes1.oslo.sysco.no
- `hostnamectl set-hostname kubernetes1.oslo.sysco.no`
- update host name in the file /etc/hostname to `kubernetes1.oslo.sysco.no`


### Configure host files

We are using the below configurations so that each of the hosts in the cluster can communicate with each other. We have copied the below configurations in our /etc/hosts file on **each server**.
```
192.168.37.48 kubernetes1.oslo.sysco.no
192.168.37.49 kubernetes2.oslo.sysco.no
192.168.37.50 kubernetes3.oslo.sysco.no
```
After applying above settings, you should be able to ping other servers from each host. For example, I can ping kubernetes2.oslo.sysco.no from the host kubernetes1.oslo.sysco.no
```
[root@kubernetes1 ~]# ping kubernetes2.oslo.sysco.no
PING kubernetes2.oslo.sysco.no (192.168.37.49) 56(84) bytes of data.
64 bytes from kubernetes2.oslo.sysco.no (192.168.37.49): icmp_seq=1 ttl=64 time=0.460 ms
64 bytes from kubernetes2.oslo.sysco.no (192.168.37.49): icmp_seq=2 ttl=64 time=0.229 ms
64 bytes from kubernetes2.oslo.sysco.no (192.168.37.49): icmp_seq=3 ttl=64 time=0.205 ms
64 bytes from kubernetes2.oslo.sysco.no (192.168.37.49): icmp_seq=4 ttl=64 time=0.199 ms
64 bytes from kubernetes2.oslo.sysco.no (192.168.37.49): icmp_seq=5 ttl=64 time=0.262 ms
--- kubernetes2.oslo.sysco.no ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4000ms
rtt min/avg/max/mdev = 0.199/0.271/0.460/0.097 ms
```

**Note** : Its important that you replace above settings with your IP and server alias.

References:
- [Hardware and OS requirements](https://kubernetes.io/docs/setup/independent/install-kubeadm/#before-you-begin)
### Verify that MAC address and product_uuid are unique for each host
 Kubernetes uses these values to uniquely identify the nodes in the cluster. If these values are not unique to each node, the installation process may [fail](https://github.com/kubernetes/kubeadm/issues/31).

- You can get the MAC address of the network interfaces using the command `ip link` or `ifconfig -a`. For me the MAC address are 
    - `kubernetes1.oslo.sysco.no` - `00:21:f6:8d:df:0b`
    - `kubernetes2.oslo.sysco.no` - `00:21:f6:6a:26:77`
    - `kubernetes3.oslo.sysco.no` - `00:21:f6:07:83:83`

- The product_uuid can be checked by using the command `sudo cat /sys/class/dmi/id/product_uuid`. For example the unique product ids in my case are 
    - `kubernetes1.oslo.sysco.no` - `1234EEE2-0002-1000-9481-2C9B3620A4FF`
    - `kubernetes2.oslo.sysco.no` - `1234EEE2-0002-1000-A7D1-ACE30E89E55F`
    - `kubernetes3.oslo.sysco.no` - `1234EEE2-0002-1000-0ED0-F6750310F8CC`

**Note** : The product ids are changed in the above example. You should see the similar pattern for your setup. Its important that these UUIDs are unique for each node that forms a cluster.

References:
- [MAC and UUID Spec](https://kubernetes.io/docs/setup/independent/install-kubeadm/#verify-the-mac-address-and-product-uuid-are-unique-for-every-node)

### Configure OS level settings
- Enable br_netfilter Kernel Module: The br_netfilter module is required for kubernetes installation. Enable this kernel module so that the packets traversing the bridge are processed by iptables for filtering and for port forwarding, and the kubernetes pods across the cluster can communicate with each other.

Run the command below to enable the br_netfilter kernel module.
```
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```
- Disable SWAP : 
    - Disable SWAP for kubernetes installation by running the following commands.```swapoff -a```
    - And then edit the '/etc/fstab' file. Run ```vim /etc/fstab``` and comment the swap line UUID
    ```
    # /etc/fstab: static file system information.
    #
    # Use 'blkid' to print the universally unique identifier for a
    # device; this may be used with UUID= as a more robust way to name devices
    # that works even if disks are added and removed. See fstab(5).
    #
    # <file system> <mount point>   <type>  <options>       <dump>  <pass>
    # / was on /dev/nvme0n1p2 during installation
    UUID=47371a4c-3cdf-4ecc-b573-f09a029c74f4 /               ext4    errors=remount-ro 0       1
    # /boot/efi was on /dev/nvme0n1p1 during installation
    UUID=C176-4947  /boot/efi       vfat    umask=0077      0       1
    # swap was on /dev/nvme0n1p3 during installation
    # COMMENT BELOW LINE
    # UUID=78aeec44-b1af-4720-9316-6af68385b23f none            swap    sw              0       0
    ```
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
