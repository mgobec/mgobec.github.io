---
title:  "What's new in Apache Cassandra 3.0 - part 2"
date:   2016-06-14 09:15:24 +0200
---
![an image alt text]({{ site.baseurl }}/images/2000px-cassandra_logosvg.png "What's new in Apache Cassandra 3.0")

In the [part one][part-one] of “What’s new in Cassandra 3.0” I got into details about materialized views and in this part I would like to touch up on storage engine rework, token allocation algorithm and a few features from 2.2 that are really useful.

#### **Storage engine rework**

Conceptually, the storage layout hasn’t changed and the initial so called map of maps remains but the major difference is that the previous Map<byte[], Map<byte[], Cell>> is now a Map<byte[], Map<Clustering, Row>>. The table itself remains the same map of partitions which are then a sorted map of Clustering and Row. The previous implementation of storage engine stored data as cells with no explicit row structure and required additional processing when re-creating rows from cells with inefficient use of disk space. The new storage engine is still cell based but it stores schema with data and column names once per partition, which reduces disk usage and simplifies the processing of rows. For a typical table definition, the difference in size looks really impressive.

![an image alt text]({{ site.baseurl }}/images/datasize_regular.png "Data sizes compared")

The old format wasn’t related to the CQL as much since it was a direct match for Thrift API, so there was a problem with representing the schema with such a simple internal structure. The row was actually a collection of cells and the storage engine wasn’t aware of their logical relation. One of the issues with such implementation was that deleting data required a ranged tombstone which is hard to evict. With the new approach, we have a specific implementation of tombstones for row and collection which take less storage space and are easier to evict.

By persisting the schema internal data is being serialized in a type-aware fashion and the result is a much compact storage model but easily expandable by new features.

#### **Token allocation algorithm**

During my time with Cassandra, I have observed some reasonable disproportions in data size and minor differences in load between nodes in the cluster. Most of this was due to the way the token was generated and no matter how you changed the topology of the cluster the same problem remained with the same nodes. The new token allocation algorithm makes sure that the new nodes take away responsibility from the nodes that are most heavily utilized.

For example, depending on the cluster size and data access patterns, with initial implementation we noticed that some nodes were being utilized more or less than average in the cluster. Adding more nodes to the cluster never solved this issue because of the initial implementation of allocation algorithm and eventually resulted in over or under utilization. Taking the number of vnodes and replication factor into consideration, this algorithm became even more complex when adding new nodes into the cluster and increasing the number of vnodes. In 3.0 version a new property was added to configuration file that specifies the keyspace for which the new algorithm will find different factors that impact the optimization process. The new nodes will still get new tokens but they will take over some token ranges of the most utilized nodes and even out of the cluster. If you are interested in more technical details you should check out this [JIRA][vnode-allocation-jira] ticket.

There were also a few interesting additions in Cassandra version 2.2 which I felt like they should be a part of Cassandra 3.0 but got released earlier because of the storage engine revamp.

#### **JSON**

One of them is **JSON** support which enables storing and retrieving of JSON documents directly from database. Schema is still enforced and data is being validated against defined types but this enables easier integration where JSON format is used for communication and is a great fit for microservice architecture.

For example, let’s create a user table defined like this:

{% highlight sql %}
CREATE TABLE user (
   username text,
   password text,
   email text,
   company text,
   PRIMARY KEY (username)
);
{% endhighlight %}

Inserting a user into the table using JSON is easy as this:

{% highlight sql %}
INSERT INTO user JSON '{"username": "mgobec", "password": “123”, "email": "matija.gobec@smartcat.io", “company”:”SmartCat”}';
{% endhighlight %}

There is no need to serialize JSON into application entity before inserting data.

Select statement would look like this:

{% highlight sql %}
SELECT JSON * FROM user;
{% endhighlight %}

And the output of cqlsh SELECT statement will return something like this:

{% highlight sql %}
[json]
-------------------------------------------
{"username": "mgobec", "password": “123”, "email": "matija.gobec@smartcat.io", “company”:”SmartCat”}
{% endhighlight %}

Select returns a single column named `[json]` with a JSON-encoded map representation of the row. One catch when working directly with JSON is that INSERT and SELECT statements work with the entire row. When you want to select or insert a single column, you need to use toJson() and fromJson() functions.

This can be considered a benefit since we can work directly with JSON and there is no need to serialize the data to and from JSON on the application level. Serialization is done on the cluster side and it does marginally impact performance compared to querying raw data, but the overall results from the client’s perspective are better when compared to converting rows to application entities and then converting them to JSON. I think it’s a good feature that enriches the list of possibilities of Cassandra database.

#### **Performance and security**

A few changes worth mentioning are performance improvements of the commit log and commit log compression which is turned on by default. AWS deployments got performance improvement by network message coalescing, together with the addition of role-based access control.

#### **The future is bright**

With the 3.0 version and the updates that were pushed with 2.2, the people developing Cassandra did a really good job. I’m not a huge fan of all the features but I love performance improvements and the whole storage engine rewrite. Things like JSON support, materialized views, UDFs and aggregates are mostly added for simplicity of use in certain use cases and to enable some corner cases that were hard to implement before. I’m looking forward to the new releases and the so long awaited SASI index adoption.

[vnode-allocation-jira]: https://issues.apache.org/jira/browse/CASSANDRA-7032
[part-one]: {{ site.baseurl }}{% post_url 2015-12-21-whats-new-in-apache-cassandra-3-0-part-1 %}
