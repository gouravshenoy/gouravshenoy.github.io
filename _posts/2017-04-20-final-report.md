---
layout: post
title:  "Science Gateway Architectures: Final Report"
date:   2017-04-20 20:00:00
categories: apache-airavata spring17
tags: workload databases microservices event-driven
---
## Final Report for Science Gateways Course!
For the past two weeks, I have been working on the following work-items:
1. Continuing the work on event-driven database replication for Apache Airavata microservices.
2. Working with Ajinkya and Amruta on the Distributed Task Execution project.

## Event Driven DB Replication
In my [previous blog](/apache-airavata/spring17/2017/03/23/akka-dbevent-workload.html), I mentioned about using ```topic``` based implementation of RabbitMQ for delegating events (messages) from a publisher service to a subscriber service. Note here, a publisher service is one which will forward the db replication request, and subscriber service is one which will have it’s db replicated via events. We have finally completed the implementation of this architecture (see image below) for Apache Airavata Services, created a Pull-Request and got it merged to the ```apache/airavata``` repository.

**Updated architecture using TOPIC based implementation**
<img src="/images/event-driven-db-topic-based.png" alt="Event Driven DB Replication Architecture" style="height:70%;width:100%" />

**How it works?**
* The db-event-manager listens to the ```db.event.queue``` queue, and also have a connection with a zookeeper ensemble.
* If a service needs to be replicated (subsciber-service), it will send a message to the ```db-event-exchange``` with a routing key ```db.event.queue```. This message will contain details about the publisher-service it is interested in (source of data replication). The exchange will forward this message to the db-event-manager.
* The db-event-manager will save a mapping to the zookeeper via hierarchical nodes.
* When the publisher-service updates any entity, it will send a message to ```db-event-exchange``` with a routing key ```db.event.queue```. This message will contain the data model of the entity (for replication), along with the type of CRUD operation.
* The db-event-manager will then lookup zookeeper for a publisher-subscriber mapping and accordingly forward the message to respective subscriber-service(s), which will then update it’s database.

**Illustration**
<img src="/images/db_event_routing.png" alt="Event Driven DB Replication Example" style="height:70%;width:100%" />

We have made a small implementation change to accommodate the possibility of failures resulting from routing messages (events) to multiple exchanges. Now, instead of routing events to multiple exchanges (for each unique service), we will use a single exchange and take advantage of RabbitMQ’s routing key naming nomenclature to redirect messages to multiple queues. For example, in image above, we can construct a routing key ```service_B.service_C.service_D```, which will make sure that a single message is delivered to multiple queues of respective subscriber services. With this new tweak, we just need one publisher at DB event manager and messages can be sent by altering routing keys. 

## My Git Commits
* **Event-Driven Database replication**: I am currently commiting my code to Ajinkya's forked ```ajinkya-dhamnaskar/airavata``` repository. I then create Pull-Requests to get it merged into the main ```apache/airavata``` repository.
	* Commits made to Ajinkya’s forked repository can be [tracked here](https://github.com/ajinkya-dhamnaskar/airavata/commits/user-profile?author=gouravshenoy).
	* Commits merged to main Airavata repository can be [tracked here](https://github.com/apache/airavata/commits/user-profile?author=gouravshenoy).

## Airavata JIRA Entry
* [JIRA LINK 1](https://issues.apache.org/jira/browse/AIRAVATA-2334)
* [JIRA LINK 2](https://issues.apache.org/jira/browse/AIRAVATA-2338)


## Airavata Pull Requests Created
* PR 1: [Add Profile-Service SDK and Implement Event-Driven DB Replication](https://github.com/apache/airavata/pull/105).
