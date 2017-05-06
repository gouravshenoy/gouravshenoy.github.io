---
layout: post
title:  "Science Gateway Architectures: Final Report"
date:   2017-04-20 20:00:00
categories: apache-airavata spring17
tags: workload databases microservices event-driven
---
## Final Report for Science Gateways Course!
For the past few weeks, I had been working on the following work-items:
1. Finishing up with the Distributed Task Execution class project with Ajinkya and Amruta.
2. Implementing a distributed task execution framework for Apache Airavata.

## Distributed Task Execution Class Project
We have been able to successfully implement and demonstrate a working prototype of this framework (implemented in Java). This [blog](/airavata-courses/spring17/2017/02/10/distributed-workload.html) here gives a gist of the prototype we considered for implementing this framework. The subsequent sections explain in detail the final contributions to this project. These include graph-database integrations (NEO4J), using Zookeeper for experiment recovery, containerizing the microservices (Docker), and deploying it on a distributed container orchestration platform (DC/OS).

**Graph Database (Neo4j) Integration**
As presented in earlier blogs, an essential feature of our framework is the ability to create and maintain task execution workflows as ```Directed Acyclic Graphs (DAGs)```. In order to do so efficiently, we need a graph database which can handle distributed loads and yet be inherently fault tolerant. Therefore we decided to go ahead with [Neo4j](https://neo4j.com/developer/). Further, we have used the graph database to store 2 types of DAG information:
1. an abstract version of the DAG (for each application-type), which specifies the task workflow for different type of applications.
2. a loaded DAG (for each experiment), which is constructed on experiment creation, with metadata needed for that task to execute.

The image below depicts the structure of a node in the DAG. 
<img src="/images/node-structure.png" alt="GraphDB DAG Node" style="height:50%;width:50%" />

When experiment is submitted, based on application type, the DAG is created and stored in the graphdb (neo4j) along with other metadata. The ```experiment-id``` is used as a relationship between nodes. The figure below is a graphical representation of the DAG stored in neo4j. To the right is the master DAG - which is an abstract version - which store series of tasks for different types of experiments with type being the relationship type between nodes. To the left, we have a loaded DAG - which has nodes filled with task context and other required metadata with the experiment-id being the relationship type.
<img src="/images/neo4j-dag.png" alt="Neo4j" style="height:70%;width:100%" />

**Using Zookeeper for Experiment Recovery**
We are using ```Zookeeper``` in order to recover experiments that were unable to complete due to system failure. When an experiment is launched, a node is created in Zookeeper. That node remains until the experiment successfully completes. If for some reason the system encounters a failure, and restarts, we have a experiment recovery script, which pulls in all nodes from zookeeper and resumes progress from where it last stopped. This process is simplified due to the fact that our DAG nodes have metadata to indicate if a particular task has completed or not, and therefore when the recovery script starts an experiment, it will only resume from the task that was not marked complete.

**Containerizing the system**
Our prototype framework followed a micro-service architecture implemented in Java. Therefore it was relatively easy to write ```Dockerfile``` for each service - which in turn creates Docker images for these services. Our system also uses third party applications such as ```RabbitMQ```, ```Zookeeper```, ```MySQL```, and ```Neo4J```. Since there are docker images readily available for these on Docker’s public repository, we had to just submit the ```$ docker run <name-of-image>``` command to get those services started.

We built docker images for all of our services and pushed them to [DockerHub repository](https://hub.docker.com/u/goshenoy/). Following are the docker images we built for our services:
1. goshenoy/workloadbase:2.0 - the base image which contains the maven built code.
2. goshenoy/workloadorchestrator:2.0 - the base with orchestrator component started.
3. goshenoy/workloadscheduler:2.0 - the base with scheduler component started.
4. goshenoy/workloadapi:2.0 - the base with api-server component started.
5. goshenoy/workloadworkers:2.0 - the base with task workers started (job-submission, env-setup, data-staging, monitoring).

For instructions on how to run the system locally, using Docker, please follow instructions posted on our [Wiki](https://github.com/airavata-courses/spring17-workload-management/wiki/Guidelines-on-how-to-run-the-system)

**Deploying on DC/OS Platform**
We discussed the validity, exibility and robustness of the proposed framework. Now, it is important to consider deployment aspect for the same. We chose distributed operating system also known as```Datacenter operation system (DC/OS)``` based on ```Apache Mesos``` distributed system kernel. It is a distributed infrastructure as a service which abstracts out multiple machines as if they were a single computer. It facilitates resource management, process placement, inter-process communication and simplies the installation and management of distributed services. This environment is best suitable or our framework to ensure load handling, resource utilization and fault tolerance. 

Figure: The DC/OS Dashboard for Cluster installed on AWS. The services and nodes health status is displayed.
<img src="/images/dcos-dashboard.png" alt="DCOS Dashboard" style="height:70%;width:100%" />

We have created DC/OS cluster with default template using Amazon cloud formation, which has 6 machines out of which one is public facing. The services need to be containerized before running on DC/OS. We created docker images for these services, DC/OS takes these docker images and run it on a cluster. Worker image is parameterized and can be configured to start individual or multiple tasks. Here, end users are completely unaware of machines on which different services are deployed. These services can be configured to scale up and down based on load, for that matter DC/OS shuts or spin ups instances as required and thereby saves resources and cost. DC/OS employs Health Check module to monitor services, if any of the services goes down it restarts that service transparently. In our case, multiple instances of tasks can be deployed to balance load, if any service is taking heat, DC/OS can identify the same and scale that service horizontally.

Figure: A view of the services (docker containers) installed in our DC/OS cluster.
<img src="/images/dcos-services.png" alt="DCOS services" style="height:70%;width:100%" />

**The User Interface**
The user interface for this prototype has been built using ```Bootstrap```. The UI talks to the APIs exposed by the API server. The UI has been built to make it easy to try out the framework, and currently only supports the following:
1. Submitting a new experiment by selecting type of application.
2. Real-time monitoring of the experiment, along with audit-trail of tasks.
3. Listing all experiments and filtering the list.

Figure: View of the User Interface deployed as a Java web-app container on DC/OS.
<img src="/images/distributed-tax-ex-ui.png" alt="DCOS services" style="height:70%;width:100%" />

## Distributed Task Execution for Apache Airavata
We have started the work for Apache Airavata. We have been able to successfully disintegrate the ```GFAC``` module and create individual task implementations. We are now working on the GFAC Engine and pulling out code to create the Orchestrator.

## My Git Commits
* **Class Project**: [LINK](https://github.com/airavata-courses/spring17-workload-management/commits/master?author=gouravshenoy)
* **Apache Airavata**: [LINK](https://github.com/apache/airavata/commits/feature-workload-mgmt?author=gouravshenoy)

## The Final Report (PDF)
The final PDF report in IEEE paper format has been submitted on Canvas.