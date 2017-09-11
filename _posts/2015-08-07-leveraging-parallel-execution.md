---
title:  "Leveraging parallel execution"
date:   2015-08-07 16:14:22 +0200
---
![an image alt text]({{ site.baseurl }}/images/leveraging_parallel_execution.jpeg "Leveraging parallel execution")

With NoSql databases comes change in physical data modelling. When it comes to truly distributed databases, data model is denormalized and, in case of Cassandra, based on the way we query data. The idea behind this is that each query is going to target only single partition (file) on a node and read will require only single disk seek. Sometimes, there are some cases where we need to query from multiple partitions but there is no need to create a model to support all the required data in a single partition. Cassandra query language (CQL) supports executing queries with an IN clause. This is nice to have but it's usually too much strain on a coordinator node and should be avoided in the long run. On the other hand, there are some cases where we need to query different tables thus executing multiple partition queries but without the possibility to use the IN clause. This is where parallel execution comes to play.

Cassandra datastax driver provides a great ability to execute a sequence of queries on multiple nodes and by doing that, leverage the power of a distributed system.

Imagine we have a group of people and we need to ask them a set of questions. For each question, there are 3 people in the group that know the answer. Each person knows who are the 3 that know the answer. If we give a list with all the questions to one guy who then needs to find all the people that know the answers in a sequence and then return to us with all the answers it will take some time and that one guy will have to do a lot of work.

On the other hand, if we ask each question to a different person in a round robin fashion, then a lot of questions will be answered in parallel and answer bearers will get back to us quicker and in about the same time.

When this is translated to Cassandra nodes, the first case scenario would require coordinator node (the one that we give all the questions to) to go around and query other nodes for data and then acquire all the answers, put them in a single dataset and return to us. The second scenario would be to go around the cluster and query each node with the next request in the list. This would give much better results since there are a lot more workers working on getting the requested data from replica node.

Here’s one example:
Let’s say we have a logger on gates of some stadium entrance. Each gate will track people coming in at some time and distinguish them by their ticket number.

Now imagine that this is going on every day and we are keeping all the records throughout the year for each gate. If we know that one gate would hold up a fairly small number of entrance records (10-20k) per year then it’s fine if we keep it in one partition. There are some queries that require us to have this partitioned per gate so that we could also make some analysis easier. Now there could be hundreds of gates per stadium and we want to get all that information for the whole year. Here comes the benefit of parallel queries. So, instead of making coordinator node execute all these queries for each gate for that stadium, we will execute all the queries in parallel. This drastically reduces the response time of the complete query because there is a lot happening in parallel. The driver itself provides the future handle for the result, so when the query is finished, it populates the result and we can collect the data.

Instead of querying with the IN clause and putting too much stress on a single node, we are executing multiple queries in a round robin pattern and the stress is distributed across all nodes in the system, but also the possibility to hit replica node is higher. We can instantly get a response by querying data from replica node. In a 20 node cluster with replication factor of 3 there is a 15% chance that the coordinator node will be the replica node. This is not much of a chance but it beats a single node query any time. In a larger scale system with a large number of nodes, this should be solved by using spark and cassandra spark connector which is data aware and all the queries would get executed against nodes holding the data (replica nodes).
