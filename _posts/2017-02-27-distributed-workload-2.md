---
layout: post
title:  "Implementing Distributed Workload Mgmt, and the Hackillinois Experience"
date:   2017-02-27 10:00:00
categories: airavata-courses spring17
tags: workload distributed implementation
---
# Implementing Distributed Workload Management, and the Hackillinois Experience
This blog post explains the implementation details of distributed workload management, and my experience at Hackillinois 2017 in Urbana Champaign, IL. 

## Recap
As explained in my previous [blog post](/airavata-courses/spring17/2017/02/10/distributed-workload.html), the goal is to find an efficient way to manage workload/tasks in a microservices based distributed system, with an emphasis on inheriting the idea to benefit Apache Airavata. Based on the developer mailing list discussions, we decided to follow an approach inspired by the design of popular distributed schedulers such as [Apache Mesos](http://mesos.apache.org/) and [Hashicorp Nomad](https://www.nomadproject.io/). You can more details in the [blog](/airavata-courses/spring17/2017/02/10/distributed-workload.html).

## The Worker, Scheduler and RabbitMQ
As of today, we have the following implementations ready and tested:
* The Scheduler Service
* RabbitMQ Messaging
* The Workers - task implementations, which include:
  * Job Submission Task
  * Data Staging Task
  * Environment Setup Task

I have contributed to the implementation of the Job Submission Task, Scheduler, and RabbitMQ messaging. The current job submission assumes only cloud job submission. I have setup a Mesos cluster with 1-master and 1-slave on Amazon using spot instances. This Mesos cluster uses Apache Aurora as the job scheduler, and hence the job-submission task implemented will be using aurora thrift client to interact with this cluster.

The cluster details are as follows:
* Mesos dashboard: [MESOS LINK](http://54.152.106.52:5050/#/)
* Aurora dashboard: [AURORA LINK](http://54.152.106.52:8081/scheduler/)

Each of these implementations make use of RabbitMQ messaging queues(each have their respective queues) to communicate with each other. For messaging we decided to go with the worker model, since it guarantees message consumption by single worker at a time. If worker fails to process, the message gets queued again for other waiting workers. This messaging infrastructure also supports priority scheduling, as Ajinkya and me, we discussed the need to have the support for prioritized scheduling.

The current scheduler does not have custom logic to decide which worker will serve the request, i.e. it does not perform match-making on it’s own. Rather, we are using messaging for the scheduling purpose. We have been discussing [this scheduling issue of PUSH vs PULL](https://github.com/airavata-courses/spring17-workload-management/issues/6), and I feel it is subjective; the current design is PULL oriented, we will be exploring the PUSH paradigm next.

For testing purpose we have written unit-tests for each feature. The basic flow for testing is as follows:
1. Build the aurora-client, workload-commons, and RabbitMQ projects.
2. Build the Scheduler, JobSubmissionTask, and DataStaging projects.
3. This will generate JARs in each project above (2).
4. Start each JAR using the ```$ java -jar``` command, which will essentially start the RabbitMQ subscribers for each project.
5. Test the end-to-end flow using a mock Orchestrator in the ```SchedulerTest.java``` file in the Scheduler project.

## Gossip protocol using SERF
I have setup [SERF](https://www.serf.io) on the worker and scheduler nodes to perform cluster membership using gossip protocol. The purpose of using SERF is to enable the following:
* If a worker joins/leaves/crashes, the scheduler is informed and can take some action.
* Workers can communicate their attributes such as task execution capabilities, system performance, etc to the Scheduler. This will help the Scheduler make informed decisions such as which worker to forward the task execution request.

Currently I have configured the Scheduler SERF agent to send an email to admin (myself) via handlers, in case any of the workers go down.

## Github Commits
All commits are currently being made to the **develop** branch. My GitHub commits can be [tracked here](https://github.com/airavata-courses/spring17-workload-management/commits/develop?author=gouravshenoy).

## Github Issues
I have created Github issues to track the progress of implementation and to resolve any design conflicts via discussions. The issues can be found [HERE](https://github.com/airavata-courses/spring17-workload-management/issues)

## Hackillinois 2017
I had the opportunity to mentor graduate and undergraduate students at Hackillinois 2017. The idea was to guide the students in solving some challenging problems related to Apache Airavata. We had a bunch of brilliant and amazing students approach us - some were completely new to Distributed and Cloud computing and wanted to get their hands dirty while still learning the core concepts. We put forward problems related to devops, user-interface, and box-for-airavata; but nearly all of them wanted to contribute towards the distributed workload management problem.

They decided to break things up and work on each component. They came up with a scheduling algorithm using min-cut-max-flow which helps determine which worker needs to perform the task given n-workers and m-tasks (m >= n). They did a fantastic job, right from understanding the problem, coming up with solutions and implementing them. Their code has been merged to the **hackillinois** branch of the airaveata-courses/spring17-workload-management repository. It can be [found here](https://github.com/airavata-courses/spring17-workload-management/tree/hackillinois). Overall it was a wonderful experience, and I had this unique opportunity of learning from the students as well as imparting the knowledge of cloud computing and opensource contribution.


