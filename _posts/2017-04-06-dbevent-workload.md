---
layout: post
title:  "Distributed Task Execution & Event-Driven DB Management Contd."
date:   2017-04-6 20:00:00
categories: apache-airavata spring17
tags: workload databases microservices event-driven
---
## New artifacts explored, and progress report so far!
For the past two weeks, I have been working on the following work-items:
1. Learning distributed application development using the 'Actor' model, with [AKKA](https://akka.io).
2. Continuing the work on event-driven database replication for Apache Airavata microservices.
3. Working with Ajinkya and Amruta on the Distributed Task Execution project.
4. Wrote a paper on 'Distributed Task Execution Framework' and submitted to the [PEARC'17](http://www.pearc.org) conference.

## Event Driven DB Replication
In my [previous blog](/apache-airavata/spring17/2017/03/09/event-driven-db.html), I mentioned about using ```queue``` based implementation of RabbitMQ for delegating events (messages) from a publisher service to a subscriber service. Note here, a publisher service is one which will forward the db replication request, and subscriber service is one which will have it’s db replicated via events. But after brainstorming, we realized that a ```topic``` based implementation of RabbitMQ would fit our use-case well, and also eliminate the necessity to create multiple queues at the ```db-event-manager``` side. Better, by altering a few configurations and lines of code, we can mimic the ```worker-queue``` functionality as well.

**Updated architecture using TOPIC based implementation**
<img src="/images/event-driven-db-topic-based.png" alt="Event Driven DB Replication Architecture" style="height:70%;width:100%" />

**How it works?**
* The db-event-manager listens to the ```db.event.queue``` queue, and also have a connection with a zookeeper ensemble.
* If a service needs to be replicated (subsciber-service), it will send a message to the ```db-event-exchange``` with a routing key ```db.event.queue```. This message will contain details about the publisher-service it is interested in (source of data replication). The exchange will forward this message to the db-event-manager.
* The db-event-manager will save a mapping to the zookeeper via hierarchical nodes.
* When the publisher-service updates any entity, it will send a message to ```db-event-exchange``` with a routing key ```db.event.queue```. This message will contain the data model of the entity (for replication), along with the type of CRUD operation.
* The db-event-manager will then lookup zookeeper for a publisher-subscriber mapping and accordingly forward the message to respective subscriber-service(s), which will then update it’s database.

**Illustration**
<img src="/images/db-event-example-topic-based.png" alt="Event Driven DB Replication Example" style="height:70%;width:100%" />

The figure above illustrates this process, where ```service-c``` and ```service-b``` are subscribed to db changes in ```service-a```. 

## Using AKKA for performing event-driven-db replication
Akka is a toolkit and runtime for building highly concurrent, distributed, and resilient message-driven applications on the JVM. To understand akka, you primarily need to understand the Actor model. Actors are units of computing, and you communicate with them through messages. Each actor has some local business logic, but it's more or less a black box to outside code. You can send messages to an actor, or receive a message from an actor, but you can't access it's internal logic directly. Each actor has a mailbox which is both a queue and a traffic cop.  Messages are sent by other actors, and can accumulate in the queue, but exactly one message can enter an actor's logic block. That means the code inside the actor is protected from concurrency issues. 

If you need parallel processing, you will most likely need a pool of actors that can process messages addressed to a single mailbox.  When you use a pool of actors, you can have a single mailbox serviced by multiple actors, but still only one message can enter any given actor. Since communication is done via asynchronous messages, you can scale by shaping your message queues, adding more actors to your pools , or controlling back-pressure (how much mail gets delivered). Actors are generally cheap, and it's not uncommon for applications to have hundreds of not thousands of actors at any given time.

**Possible Solution using Akka**
<img src="/images/akka-approach.png" alt="Alternative solution using Akka" style="height:70%;width:100%" />

In our case, ```db-event-manager``` and other Airavata micro-services both can be considered as actors. The publisher-service will need to send a message to the db-event-manager actor and db-event-manager actor in turn sends messages to corresponding actors based on publisher-consumer mapping saved in Zookeeper. We are still exploring Akka, and identifying good use cases. There are however some challenges, if we decide to use Akka, such as remote actor discovery, actor failure, multiple instances of the same actor(service), etc. We will be doing some more reading about Akka, but in the meanwhile, I have kick-started Akka application development using simple examples (see GitHub link below).

## My Git Commits
* **Playing around with Akka**: The commits are currently made to my personal repository; it can be [tracked here](https://github.com/gouravshenoy/akka-playground/commits/master?author=gouravshenoy). 
* **Distributed task execution**: The commits for this work can be [tracked here](https://github.com/airavata-courses/spring17-workload-management/commits/develop?author=gouravshenoy).
* **PEARC’17 paper**: The commits made for this purpose can be [tracked here](https://github.com/SciGaP/paper-pearc17-distributed-task-execution/commits/master?author=gouravshenoy).
* **Event-Driven Database replication**: I am currently commiting my code to Ajinkya's forked ```ajinkya-dhamnaskar/airavata``` repository. I then create Pull-Requests to get it merged into the main ```apache/airavata``` repository.
	* Commits made to Ajinkya’s forked repository can be [tracked here](https://github.com/ajinkya-dhamnaskar/airavata/commits/user-profile?author=gouravshenoy).
	* Commits merged to main Airavata repository can be [tracked here](https://github.com/apache/airavata/commits/user-profile?author=gouravshenoy).

## Airavata JIRA Entry
* [JIRA LINK 1](https://issues.apache.org/jira/browse/AIRAVATA-2334)
* [JIRA LINK 2](https://issues.apache.org/jira/browse/AIRAVATA-2338)


## Airavata Pull Requests Created
* PR 1: [Create Profile-Service for Apache Airavata](https://github.com/apache/airavata/pull/97).
* PR 2: [Create Tenant-Profile-Service for Apache Airavata](https://github.com/apache/airavata/pull/99).
* PR 3: [Create Profile-Service-Commons and Profile-Gateway](https://github.com/apache/airavata/pull/101).
