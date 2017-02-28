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
* The Workers - task implementations, which include:
  * Job Submission Task
  * Data Staging Task
  * Environment Setup Task

I have contributed to the implementation of the Job Submission Task, Scheduler, and RabbitMQ messaging. The current job submission assumes only cloud job submission. I have setup a Mesos cluster with 1-master and 1-slave on Amazon using spot instances. I have also installed Apache Aurora 
## Gossip protocol using SERF

## Github Commits

## Github Issues

## Hackillinois 2017



