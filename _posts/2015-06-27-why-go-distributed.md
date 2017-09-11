---
title:  "Why go distributed"
date:   2015-06-27 19:10:34 +0200
---
![an image alt text]({{ site.baseurl }}/images/why-go-distributed.jpeg "Why go distributed")

#### **Why go distributed?**

When talking to other fellow engineers and people in our industry this question appears inevitably. We have had a single instance applications for decades and there have been no problems. Scaling was done by adding another server behind a router and databases would be deployed on SAN for disk drive failure protection and that’s it. It has been doing the job for years, so why change now?

Let’s start from the beginning. The number of people in the world hasn’t grown drastically in the past two or three decades, but the number of people connected to the internet has, but that is not the fact that has had the greatest impact. What really changed is the number of devices. A few years ago every third person had a device capable of collecting and sending any kind of data to some service. Now we have at least three of these devices per person. Smartphones are one of the biggest sources of data out there. Each phone generates huge amounts of data and at this point there are around 6 billion of smartphones online. But that’s not all. Today we have smart watches, smart freezers, smart tvs, even the cars are smart. Everything around us is either already connected to the internet or is going to be in the next few years. Don’t get me wrong, I really think that’s an awesome thing. There are quite a few discussions about personal data and privacy and how this will all make it easy for hackers to get our information and gain control over our lives, but looking at the bright side, we will have so much more information to help us improve our everyday life. With so much data going “through the wire” we can run all kinds of analysis and get incredible answers.

#### **But why?**

Now, looking at the question about distributed systems we started with, the answer is shaping up. There is no way a single computer or multiple instances of one can handle and store even a fraction of this huge data flow and execute complex algorithms and maybe even learn along the way. Distributed systems are created out of need and today's technologies are getting better and better at being distributed. When speaking about hardware itself, today we have multicore servers and hundreds of megabytes of ram and a couple of terabytes of storage and it's really amazing, but the problem is that, in today's big data world, we could easily need thousands of cpu cores, terabytes of ram and even petabytes of storage space. This is not something that can be solved by a single server or even multiple instances of it. The solution lies in distributing this data across multiple machines that work in a coordinated system. There is a great line from Randy Shoup from eBay: “If you can’t split it, you can’t scale it”.

#### **Architecture**

There are two types of distributed systems by architecture. The first and easier one is master slave configuration and the second one is masterless. In a master slave architecture, there is one computer in the system that executes administrative tasks and coordinates the other computers in the system. This is relatively easy to implement because all the synchronization is centralized and all participants get orders from and report to one leader. The downside is that this is a single point of failure in the system which is supposed to help us solve this issue. There are some mechanisms that overcome this problem like having a spare master with data replication or all the participants in the system voting for one of them to become the new master.

In a masterless architecture all the participants are equal and they communicate to all other participants in the system. This leads to easier failure tolerance implementation since nothing is shared but the downside is keeping everything in sync and timely recognition of failures. There is no master to keep an eye on everything so the participants need to talk to each other a lot. Working with both types of systems from a developer's standpoint are both hard to start with. There is so much going on and it’s hard to debug problems on a single computer, not to mention three or more.

#### **Where to start**

The easiest way to start with development is by setting up a cluster of 3 of each components in the system. This shouldn’t be a problem with tools that are available today like ansible, vagrant, docker or others. By creating a distributed development environment, it’s much easier to understand how it all works and problems with certain approaches become visible in early phases. The worst thing you can have is simple bugs wreaking havoc on deployed system which you could easily pinpoint and solve during development. This is especially important for masterless distributed systems with loose consistency.

The second most important thing is logging. People developing software usually tend to start with development and then add logging along the way or as it becomes necessary. It should be a good practice for any type of development to set up proper logging first, as it’s always hard to add it later. Also, it is a good idea to use one of the online logging services like loggly or rollbar because it’s easy to setup and monitor and makes life much easier.

#### **My two cents**

Distributed systems are a great representative of “everything comes at a price”. They solve a lot of impossible cases for a traditional system and bring a lot of gains but at the same time they introduce problems of their own. If you understand what you are doing and setup all the tools and logging properly there is nothing to be afraid of.
