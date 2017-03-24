---
layout: post
title:  "Exploring Akka, Continuing Distributed Task Execution & Event-Driven DB Management"
date:   2017-03-23 19:00:00
categories: apache-airavata spring17
tags: akka workload databases microservices event-driven
---
# New artifacts explored, and progress report so far!
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
<img src="/images/db-event-example-topic-based.png" alt="Event Driven DB Replication Example” style="height:70%;width:100%" />

The figure above illustrates this process, where ```service-c``` and ```service-b``` are subscribed to db changes in ```service-a```. 

## My Git Commits
**Playing around with Akka**: The commits are currently made to my personal repository; it can be [tracked here](https://github.com/gouravshenoy/akka-playground/commits/master?author=gouravshenoy). 

**Event-Driven Database replication**: I am currently commiting my code to Ajinkya's forked ```ajinkya-dhamnaskar/airavata``` repository. I then create Pull-Requests to get it merged into the main ```apache/airavata``` repository.
* Commits made to Ajinkya’s forked repository can be [tracked here](https://github.com/ajinkya-dhamnaskar/airavata/commits/user-profile?author=gouravshenoy).
* Commits merged to main Airavata repository can be [tracked here](https://github.com/apache/airavata/commits/user-profile?author=gouravshenoy).

**Distributed task execution**: The commits for this work can be [tracked here](https://github.com/airavata-courses/spring17-workload-management/commits/develop?author=gouravshenoy).

**PEARC’17 paper**: The commits made for this purpose can be [tracked here](https://github.com/SciGaP/paper-pearc17-distributed-task-execution/commits/master?author=gouravshenoy).

## Airavata JIRA Entry
* [JIRA LINK 1](https://issues.apache.org/jira/browse/AIRAVATA-2334)
* [JIRA LINK 2](https://issues.apache.org/jira/browse/AIRAVATA-2338)


## Airavata Pull Requests Created
* PR 1: [Create Profile-Service for Apache Airavata](https://github.com/apache/airavata/pull/97).
* PR 2: [Create Tenant-Profile-Service for Apache Airavata](https://github.com/apache/airavata/pull/99).
* PR 3: [Create Profile-Service-Commons and Profile-Gateway](https://github.com/apache/airavata/pull/101).
