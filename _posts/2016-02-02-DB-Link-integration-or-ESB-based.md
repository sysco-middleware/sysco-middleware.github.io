---
layout: post
title: DB Link integration or ESB based
categories: soa
tags: [Oracle, Enterprise Service Bus, SOA, Architecture]
author: dalibor
keep: no
---

# Pros. and Cons.
As the state owned electric company of one East European country is undergoing transformation to conform to EU standards it has been transformed from one monolith organization to different smaller entities, each with his own task. Furthermore these entities have been further subdivided based on a region. Also the new players arrived on the market as the law has ben changed to accommodate that possibility. Now the different organizations that emerged from this dividing and subdividing have not been able to use a single monolith ERP and billing application to process and accommodate all possible use cases that the process of selling, producing, provisioning, and supplying electric current, requires. Different companies have been allowed to use their own IT systems to satisfy their business needs. IT systems have become heterogeneous. However because of the need for cooperation and data sharing between these organizations as both required by law and by business needs, numerous use cases requiring integration have emerged. Organizations are faced faced with dilemma whether to invest in ESB based integration solution or use, from his point of view, simpler and chipper solution. Point-to-Point Solution that is based on numerous database links between different heterogeneous databases.

## Integration entities
Beside the needs to exchange data between two organizations doing business in different regions, most of the data exchange occurs between entities doing cooperative efforts from selling to provisioning of electric current to the end users. First of all the end user has open possibility to buy electric current from different sellers based on the different package offering provided by seller. On the other hand seller has the possibility to choose from which producer to buy electric current based on the current market prices for electric production and future trend analyses. That means that distribution channel from the producer to the end user does not have to be constant as producer, provisioning channel from producer to distributor and supplier, and in some cases also supplier himself might get changed during the duration of the contract period. Numerous use cases are needed to satisfy previously mentioned requirements and we are going to mention end explain some of them and present them in both use case UML diagrams and possible implementation outcomes.

## Use cases
From the moment when the end-user signs contract with the supplier to the moment when the electric current in delivered to the end user there are several basic use cases.

### Use case 1: Sending user data to supplier
As the seller of the electric current does not have to be business that is directly related with provisioning and distributing of the electric current, electric current supply can be part of the bigger packages that can include e.g. telephone plus Internet plus TV plus electric current in the same package. Moreover these packages can be sold by intermediary agency that is just selling previously mentioned packages from different suppliers. In the case of the agency selling supplier packages the contract data e.g. user info should be exchanged with Supplier Company. This data includes as minimum: user first and last name, tax number, address and measurement point identification number on the current location, if exist.

![](/images/2016-02-02-DB-Link-integration-or-ESB-based/01.png)

Fig. 1. Sending data to supplier use case

### Use case 2: Checking the existence of measurement point ID number in distributors databases
After the user data from the contract have been stored in the supplier database the ID number of the measurement points and the owner is verified within the databases of all the distributor covering given location of the measurement point. If the owner of the measurement point is not the same as the signer of the contract, the owner resolution use case has to be started.

![](/images/2016-02-02-DB-Link-integration-or-ESB-based/02.png)

Fig. 2. Checking measurement point ID use case

### Use case 3: Sending the order for installing new measurement point for new users to the chosen distributor
If the new customer does not have measurement point installed and registered, the order for installing new equipment has to be given to the distributor. If there are multiple distributors for the given area the preferred one has to be chosen. This can be done based on price or some other criteria. When the distributor has been chosen the order for creating another user measurement point will be created.

![](/images/2016-02-02-DB-Link-integration-or-ESB-based/03.png)

Fig. 3. Sending the order for installing new measurement point use case

### Use case 4: Receiving monthly measurement point data from the distributor
Every month the measurement point data has to be sent from the distributor to the supplier for billing process to run. The measurement point ID number is key identification in retrieving this data.

![](/images/2016-02-02-DB-Link-integration-or-ESB-based/04.png)

Fig. 4. Receiving monthly measurement point data use case

## Possible solutions
Of course that use cases previously described are very simplified and the real electric distribution and supplying system requires many more use cases with more complex data interaction. However this suffice to stress the point in possible solutions to realize them.

### Solution 1: DB link based integration
DB link is traditional Point-to-Point integration approach that does not use central point of message distribution or message bus. It is based on proprietary technology to establish DB link.

![](/images/2016-02-02-DB-Link-integration-or-ESB-based/05.png)

Fig. 5. Possible deployment diagram of DB Link solution to realize previously described use cases

As we can see from figure 5, distributor 1 has already web application running on Weblogic server which can expose web service interface very easily but in the same time web application uses Oracle database data for normal operation and therefore can update intermediary table used in DB Link communication. Other distributors are using client server applications without web service capabilities so for them DB Link seams like natural solutions

#### Advantages:
- For old style developers DB Link is easier to implement. They do not require additional knowledge of using integration technologies and functionality of message bus.
- Communication channel is more reliable as it is binary oriented around TCP sockets and usually faster.
- There is no need for data transformation from binary to text and back and there is no need for data validation.
- Communication is statefull and both parties are immediately notified when the communication channel is broken.
- Security implementation is simplified.

#### Disadvantages:
- Implementation lacks flexibility and that's means that whenever data set changes in both elements and types the intermediary tables have to be adjusted and whole process has to be redone.
- Number of parallel communication channels is severely limited.
- Additional ports beside standard web ports need to be opened in firewall and trough these ports internal LAN is opened for hacker attacks.
- Database credentials are sent over WLAN and that exposes the network for Man in The Middle attacks to database.
- Message recovery process on error, and link reestablishment has to be implemented manually in many cases.
- Supplier has to maintain Point-to-Point link with each Distributor separately.
- Heterogeneous links between Oracle and other databases are not so easy to create and maintain and requires using vocabulary of the database you are connecting with.
- If Distributor changes his internal IT system also Supplier has to adapt to that change.

### Solution 2: ESB based integration
Enterprise Service Bus represent central message distribution system that can be easily reconfigured whenever demands change. It is usually based on XML message exchange and XML processing. Although Distributor 2 and 3 do not have native web service capability to expose XML data, that is easily achievable inside ESB system by using connectors.

![](/images/2016-02-02-DB-Link-integration-or-ESB-based/06.png)

Fig. 6.  Possible deployment diagram of ESB solution to realize previously described use cases

In this scenario both Supplier and Distributor 1 have Web application running on WebLogic application server that can easily implement both Web Service and Web Service client interfaces.

#### Advantages:
- Allows stateless style communication with many more parallel channels and parallel processing of requests.
- Allows sending more complex data structures in hierarchical form than just table-based approach. That means that multiple table content can be send in the single request.
- Allows very easy connectivity with many different system types trough many different channels.
- Switching types of connection is very easy. That allows smooth system upgrades for connected systems.
- Connection failover and retry mechanism is build into the system.
- Overloading mechanism is built into the system.
- Allows just partial data validation.
- Data transformation from one format to another is very easily achieved and does not requires that external systems change their negotiated data format after each upgrade.
- •    Network traffic is essentially text based and uses standard web ports and that means that every system that uses some form of web service connection can pass trough proxies and firewalls effortlessly.
- ESB can hide other systems in separate subnet thus making hacker intrusions more difficult.
- ESB has fine granularity of security services allowing remapping of user credentials, digital signature checking, protection of communication on both transport and message level, etc.

#### Disadvantages:
- Requires additional license for ESB installation and additional infrastructure support.
- Requires additional processing by the engine as the data very often need to be transformed from other formats to XML and back.
- Engine work essentially in interpreted way and thus the processing speed is reduced. This can be compensated with additional parallelism and stronger hardware.
- Requires additional knowledge by both administrators and developers to maintain and develop for ESB.
- Transaction handling is a little bit more difficult than in typical database operations.

## Conclusion
With this article I have tried to present two different integration approaches. Traditional or Point-to-Point based on DB links and new more modern approach based on ESB technology. By comparing advantages and disadvantages of both approaches it is clearly visible that to integrate heterogeneous systems that connect with each other over WAN networks it is by no means more flexible, more extensible and more secure to use ESB technology than traditional Point to Point. As the systems are migrating from legacy to web oriented, this approach is even more important as it allows integrating different heterogeneous systems easily with small development effort to adopt migration of one external system from one technology to another. This is especially true when number of connected systems and number of Use Cases has tendency of increase over time. DB link approach can be used just as an in house temporary solution for integrating small number of relational database based legacy systems when the network connectivity is stabile. For all other integration requirements ESB is far better choice.
