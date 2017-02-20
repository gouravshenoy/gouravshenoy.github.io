---
layout: post
title:  "Event Driven Database Replication for Microservices"
date:   2017-02-19 18:00:00
categories: airavata-courses spring17
tags: databases microservices event-driven
---
# Event Driven Database Replication for Microservice
This report mostly contains links to Wiki pages written to address different aspects of the theme: **data management for microservices**, and are part of [this repository](https://github.com/airavata-courses/spring17-microservice-data-management). You can find and navigate through all of them [HERE](https://github.com/airavata-courses/spring17-microservice-data-management/wiki).

## Problem Statement
The goal of this project is to explore possible ways to manage data across micro-services. In a nutshell, the question at hand is whether to have a “database per micro-service?” or “shared database for all micro-services?”. The outcome would eventually be applied to address possible enhancements in Aapche Airavata.

## Possible Solutions
After reading through few blogs and articles it was clear that “generally”, it is not a good idea to maintain everything in a shared database have all micro-services access it. Rather in order to achieve portability and maintainability of both the micro-services and the database, it is a good practice to keep a micro-service data private – only takes care of data which is needed for that service. Having said that, in a distributed environment it is almost impossible to NOT have any data dependencies between micro-services (if not write, then at least read). Like for e.g.: “User Profile” micro-service contains data that is most often needed by others.
 
In such a scenario, the challenge would be to keep a micro-service data private and at the same time enable sharing following the CAP theorem. We have identified 2 possible solutions to achieve this:
* 2-phase commits
* Event driven data replication

## FIGURE: An Event-Driven Architecture
<img src="/images/event_driven_db_microservices.png" alt="Event Driven DB Replication Architecture" style="height:70%;width:100%" />

The components in the architecture:
* CustomerService - Responsible for maintaining the CustomerServiceDB database. Processes the ```createCustomer``` request by creating a new record in the Customer table.
* OrderService - Responsible for maintaining the OrderServiceDB database. Processes the ```createOrder``` request by creating a new record in the Orders table.
* CommonClient - A thrift client which is used to submit the ```createCustomer``` and ```createOrder``` calls to the CustomerService and OrderService respt.
* MessageBroker - A channel to communicate events between the CustomerService and OrderService.

## How does this work?
Taking the figure above as reference, a simple db replication scenario would be as follows:
1. The user submits a request to create customer via the CommonClient.
2. The CommonClient uses the Thrift interface to make a ```createCustomer``` request to the CustomerService.
3. The CustomerService creates a new customer record in the Customer table of the CustomerServiceDB database.
4. The CustomerService then publishes an event on the MessageBroker channel, to a topic which the OrderService is subscribed to.
5. The OrderService receives this event and creates the same customer record in the Customer table of the OrderServiceDB database.

The functionality is similar for the ```createOrder``` call made to the OrderService. 
 
## Solution Evaluations
Each of the above two solutions have their pros and cons related to consistency and availability. While 2-phase commits guarantee consistency while compromising availability, event driven data replication assures eventual consistency. Ajinkya and me, we have already implemented a prototype for the event driven data replication (see github link below). We are using a message broker (RabbitMQ) as our event-communication channel. As of now, we have identified all possible corner cases and tuned our code accordingly.

## Conclusion
It is early to conclude which solution is the best, because of two reasons:
* We have just evaluated the event-driven approach, and not the 2-phase commit.
* None of these will be **THE SOLUTION** for the generic problem of data management for microservices. It ultimately depends on the specifics of what you can compromise from CAP.

We will also need to perform some kind of load testing to see how well the event-driven approach syncs when it has received say 100 requests. Similarly, see how long it takes for the 2-phase commit approach to cope under similar circumstances. I am in the favor of **eventual consistency**, and hence favor the event-driven approach, which guarantees eventual consistency.

## Wiki for Code Instructions
Please refer to this page for detailed instructions : [LINK](https://github.com/airavata-courses/spring17-microservice-data-management/wiki/Event-Driven-DB:-Steps-to-Run-Prototype)

## Github Commits
I started by creating a new repository in my personal Github account, named [EventDrivenDBMicroservices](https://github.com/gouravshenoy/EventDrivenDBMicroservices). All code for event-driven database replication was developed by Ajinkya and Me. After this theme was formally announced in class, we created a new repository in the ```airavata-courses``` account and imported code from personal repository. Due to this import, personal commit history in ```airavata-courses``` was lost. I have provided links to both commit logs below. 

* My commits to personal repository : [LINK](https://github.com/gouravshenoy/EventDrivenDBMicroservices/commits/master?author=gouravshenoy)   
* My commits to airavata-courses repository : [LINK](https://github.com/airavata-courses/spring17-microservice-data-management/commits/event-driven?author=gouravshenoy)

## Airavata Dev Mailing List Discussions
Below are links to Apache Airavata developer mailing list discussions which I have contributed to.

* Introducing the theme : [LINK](http://mail-archives.apache.org/mod_mbox/airavata-dev/201702.mbox/%3CFAC2EB20-DE9D-466D-A803-48A42290F5C7%40indiana.edu%3E)

## References
1. [A Database-Per-Microservice Architecture.](http://microservices.io/patterns/data/database-per-service.html)
2. [An Event-Driven Solution.](http://microservices.io/patterns/data/event-driven-architecture.html)
3. [Does each Microservice really need its own database?](https://plainoldobjects.com/2015/09/02/does-each-microservice-really-need-its-own-database-2)
4. [CAP Theorem Details.](https://en.wikipedia.org/wiki/CAP_theorem)
