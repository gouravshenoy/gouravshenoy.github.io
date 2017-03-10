---
layout: post
title:  "Event Driven Database Replication for Apache Airavata"
date:   2017-03-09 17:00:00
categories: airavata-courses spring17
tags: databases microservices event-driven
---
# Event Driven Database Replication for Apache Airavata
This blog is in a practical extension to my previous one on the same topic [here](/airavata-courses/spring17/2017/02/19/event-driven-db.html). As you might have noticed, in the previous article I have mentioned the problem of data management in a micro-services based distributed system; and how we can tackle it using events (assuming we have one-db-per-microservice). The blog used a prototype Customer-Order example to explain details. But now it's time to put it to test in a real distributed application - Apache Airavata. 

## The motivation
Apache Airavata is a micro-services based distributed system which acts as a middleware for researchers to run scientific experiments on a supercomputing cluster. The current Airavata has a ```registry``` microservice which interacts with a database storing all information about different entities, such as users, gateways, applications, experiments, etc. I am now working with the Airavata team on an effort to break this ```registry``` micro-service into smaller independent micro-services such as ```profile-service``` (which will manage users, tenants, groups), ```sharing-registry```, etc. Each of these micro-services will have their own database to manage information. The catch is, an entity such as USER is most likely to be used by many other services such as ```registry``` or ```sharing-registry```. So how do we make sure these services remain updated with the latest data? i.e. When any new USER is added via the profile-service, how does the sharing-registry know about it, and use it? The answer is using an event-driven database replication.

## The event-driven design
To perform replication using RabbitMQ messages (a.k.a. events), we have two options:
1. Each micro-service will have subscribers/listeners for every entity that it is expecting to be replicated. For example, if we need to replicate USER and SHARING entities in registry, then the registry will need to have 2 subscribers/listeners/handlers, one for each queue. This leads to replication of logic, as well as the need to update a micro-service every time it expects a new entity from a new service OR is sending events to a new service (which means restarting the microservice everytime this happens - not economical).

2. The second way is to use a centralized ```db-event-manager``` which has information about events that a service is expecting; so that when a service (publisher) sends a new event, this manager can forward the message to appropriate services (subscribers). This idea to maintain a publisher-subscriber map is much more efficient and economical. A pictorial depiction of this design is given below.
 
## FIGURE: An Event-Driven Architecture
<img src="/images/airavata_event_driven_data_replication.png" alt="Event Driven DB Replication Architecture" style="height:70%;width:100%" />

## How does this work?
Taking the figure above as reference, a simple db replication scenario would be as follows:

**A application-service makes the event-manager aware of its interest in user_profile-service**
It is as simple as application-service sending a message to the event-manager with the name of user_profile-service. The event-manager then updates it's map with this information, and whenever a new event is published by user-profile, it will forward it to application-service.

**A new user is added to the system**
When a new user is added to the user_profile-service, it publishes an event with operation type(CRUD), entity type and actual entity (TBase object). The event-manager processes event queue and acknowledges back to publisher, so user_profile-service is now free to commit transaction. Now its event manager’s responsibility to deliver this event to interested services. 

## Solution Evaluation
The db-event-manager uses local map to identify corresponding subscribers, in this case sharing_sub and application_sub and sends user creation event to respective queues. Now, the challenge here is to employ permanent caching logic to avoid loss of this mapping in case event manager crashes. We are using ZK as a permanent cache to store this mapping, so when event manager comes up it can restore mapping using ZK node state. When event manager alters local map correspoding changes are also replicated in ZK.

There can be multiple instances of a same service listening to a queue but only one would react to the message as we are using work queue model. ZK creates a publisher entity node, any service which is interested in this entity will have node inside publisher entity. Here, user is a parent node (publisher entity) and all the nodes inside user are interested services(subscribers).

## Airavata Dev Mailing List Discussions
Below are links to Apache Airavata developer mailing list discussions which I have contributed to.

* Introducing the theme : [LINK](http://mail-archives.apache.org/mod_mbox/airavata-dev/201702.mbox/%3CFAC2EB20-DE9D-466D-A803-48A42290F5C7%40indiana.edu%3E)

## Airavata JIRA Entry
Please see: https://issues.apache.org/jira/browse/AIRAVATA-2338

## References
1. [A Database-Per-Microservice Architecture.](http://microservices.io/patterns/data/database-per-service.html)
2. [An Event-Driven Solution.](http://microservices.io/patterns/data/event-driven-architecture.html)
3. [Does each Microservice really need its own database?](https://plainoldobjects.com/2015/09/02/does-each-microservice-really-need-its-own-database-2)
4. [CAP Theorem Details.](https://en.wikipedia.org/wiki/CAP_theorem)
