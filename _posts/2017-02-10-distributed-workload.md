---
layout: post
title:  "Distributed Workload Management"
date:   2017-02-10 18:00:00
categories: airavata-courses spring17
tags: workload distributed
image: /images/workload_mgmt.jpg
---
# Distributed Workload Management
This report mostly contains links to Wiki pages written to address different aspects of the theme: **distributed workload management**, and are part of this repository. You can find and navigate through all of them [HERE](https://github.com/airavata-courses/spring17-workload-management/wiki).

## Problem Statement
Through this theme we intend to find the best possible solution to the issue of managing workloads, in a distributed environment with a micro-services based architecture, with emphasis on how these would benefit Apache Airavata. This leads to finding the best way that different micro-services (eg: Airavata micro-services) should communicate and distribute work. A lot has been written about the problem statement and requirements in the main [Wiki page](https://github.com/airavata-courses/spring17-workload-management/wiki).

## Possible Solutions
To begin with, we identified a proof-of-concept micro-service based example on which we could base our design discussions. Thanks to the healthy conversations on the Apache Airavata Dev mailing list, we have were able to come up with a design, and progressively improve it. We started with evaluating **state-full** vs **state-less** architecture to solve the problem. Then shifted our focus on a **decentralized** vs **centralized** state architecture, where the latter received a lot of traction and favorable feedback.

Thanks to our understanding of Apache Mesos/Aurora architecture, we were able to imbibe some fundamentals into improving our centralized-state architecture. Each of these "possible solutions" have been well documented with pictorial representations in our Wiki pages. The links to each of them are as follows.

* Proof-of-Concept Example : [Wiki link](https://github.com/airavata-courses/spring17-workload-management/wiki/Test-Example-&-Possible-Solutions)
* A state-full design : [Wiki link](https://github.com/airavata-courses/spring17-workload-management/wiki/1.-A-state-full-design-for-workload-management)
* A state-less design : [Wiki link](https://github.com/airavata-courses/spring17-workload-management/wiki/2.-A-state-less-design-for-workload-management)
* A centralized, Apache Mesos inspired design : [Wiki link](https://github.com/airavata-courses/spring17-workload-management/wiki/%5BFinal%5D-Centralized-architecture-for-workload-management)
* [KB] Messaging infrastructures : [Wiki link](https://github.com/airavata-courses/spring17-workload-management/wiki/Messaging-infrastructures)

## Solution Evaluations
Each Wiki page has a detailed analysis of the respective topic. Based on the Apache Airavata mailing list discussions ([see here](http://mail-archives.apache.org/mod_mbox/airavata-dev/201702.mbox/%3CFAC2EB20-DE9D-466D-A803-48A42290F5C7%40indiana.edu%3E)), we evaluated all possible solutions and had a conceptual agreement towards the Mesos inspired design.

## Conclusion
We have decided to start a proof-of-concept implementation of the finalized design (Mesos inspired centralized architecture). We will be creating Git issues targeting main building blocks of the design, and members are free to take up and start working on each component as they wish. There will be new Wiki pages added subsequently as we make progress. This will include instructions to build/compile, deploy, and run the code.

## Airavata Dev Mailing List Discussions
Below are links to Apache Airavata developer mailing list discussions which I have contributed to.

* Introducing workload distribution theme : [LINK](http://mail-archives.apache.org/mod_mbox/airavata-dev/201702.mbox/%3CFAC2EB20-DE9D-466D-A803-48A42290F5C7%40indiana.edu%3E)
* Define test example, state-full, and state-less design : [LINK](http://mail-archives.apache.org/mod_mbox/airavata-dev/201702.mbox/%3C5D649611-861B-46E4-ABAA-678EEE53679F%40indiana.edu%3E)
* Using a workflow micro-service : [LINK](http://mail-archives.apache.org/mod_mbox/airavata-dev/201702.mbox/%3C92C823ED-436F-4CAE-B9A3-6AB28E535BE1%40indiana.edu%3E)
* Centralized vs Decentralized design : [LINK](http://mail-archives.apache.org/mod_mbox/airavata-dev/201702.mbox/%3CCC712EF8-1BD0-4A6C-A48E-1764F67D9775%40indiana.edu%3E)
* Final design using centralized-state : [LINK](http://mail-archives.apache.org/mod_mbox/airavata-dev/201702.mbox/%3C1BD69733-6E99-4B34-B347-4667DC6A3337%40indiana.edu%3E)

## Github Issues
We have created Github issues to track the progress of implementation and to resolve any design conflicts via discussions. The issues can be found [HERE](https://github.com/airavata-courses/spring17-workload-management/issues) 
