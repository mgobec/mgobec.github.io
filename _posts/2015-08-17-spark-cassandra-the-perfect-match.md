---
title:  "Spark + Cassandra: The perfect match"
date:   2015-08-17 10:34:14 +0200
---

![an image alt text]({{ site.baseurl }}/images/the-perfect-match.jpeg "Spark + Cassandra")

Hadoop has been the leading platform for distributed data storage and analytics for some time now. It is a solid platform with a feature rich eco system. The base modules are core module Common, distributed file-system HDFS, resource management platform YARN and the much talked-about programming model for data processing MapReduce. There are also numerous packages that were built on top of Hadoop, primarily Pig, Hive and HBase.

One of the projects that started as data processing on top of Hadoop is Spark and it’s really been gaining attention since it got open-sourced. What Spark does is that it abstracts data as a distributed data collection called Resilient Distributed Dataset (RDD) and enables us to process that data using functional transformations always resulting in a new immutable RDD. What is different between Spark and Hadoop's MapReduce is that Spark does everything in memory making it much faster but also RAM dependable. Also, Spark is just a data processing framework without a storage engine.

In benchmarking HDFS and Cassandra there is a big difference in how the data is distributed and accessed so it does make a big impact on the overall performance. HDFS is generally deployed in a single data center on multiple racks but usually in the same geographical location. This is because of the way how HDFS works and how MapReduce relies on network and disk I/O executing calculations on data stored on disk. On the other hand Cassandra is deployed in a completely distributed fashion and having a cluster spanning across multiple zones and regions is actually preferred.

So we have Cassandra which is a completely distributed shared-nothing database and Spark which is also a distributed system working with in-memory data. What’s more interesting is that Spark executes transformations where the data is so the network traffic is used for passing jobs and results. With this in mind, these two technologies make a perfect couple. Both are distributed, both are resilient and both benefit from scaling so why not join the two and even deploy Spark nodes on top of Cassandra nodes.

There is a project called [spark-cassandra-connector][spark-cassandra-connector-github] which enables these two technologies to work together to get even better results. What it does is that it exposes Cassandra tables as RDDs enabling Spark to read from and write to Cassandra and also execute SQL queries. I would like to explain why the last one is really important. First, let me try to explain the problem. Cassandra has a denormalized data model which is query based. Unlike relational databases, there are no JOINs and no GROUP BYs. You cannot query data from two tables with a single query and what is also important you cannot filter data with free-style criteria. This is where one of the Spark modules SparkSQL comes into play. It’s a module for working with structured data and by using the before mentioned Spark Cassandra connector we can query data from Cassandra in a relational-like way.

Keeping in mind that connector is data location aware, this means that when we query data using SparkSQL, queries will get executed on the node containing data and only results will get passed back. This keeps the network traffic down but the response time as well.

How do you start with Cassandra and Spark ? First of all you need a running Cassandra instance or cluster, Spark and a Spark Cassandra connector libraries with all of its dependencies. The first step is to create a keyspace and table in Cassandra:

{% highlight sql %}
CREATE KEYSPACE IF NOT EXISTS testsparkcassandra WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1 };
CREATE TABLE IF NOT EXISTS testsparkcassandra.users ( username text PRIMARY KEY, name text, email text, age int );
{% endhighlight %}

Then we need to insert some demo data into that table

{% highlight sql %}
INSERT INTO testsparkcassandra.users (username, name, email, age ) VALUES ('test1', 'Test User 1', 'test.user.1@smartcat.io', 32);
INSERT INTO testsparkcassandra.users (username, name, email, age ) VALUES ('test2', 'Test User 2', 'test.user.2@smartcat.io', 22);
INSERT INTO testsparkcassandra.users (username, name, email, age ) VALUES ('test3', 'Test User 3', 'test.user.3@smartcat.io', 12);
{% endhighlight %}

Now when we have the data we need to create Spark configuration and initialize Spark context

{% highlight scala %}
val conf = new SparkConf(true)
.set("spark.cassandra.connection.host", "127.0.0.1")
.setMaster(SparkMaster)
.setAppName(SparkAppName)
val sc = new SparkContext(conf)
{% endhighlight %}

At this point we initialize spark context and we can get table rdd from cassandra

{% highlight scala %}
val usersTable = sc.cassandraTable("testsparkcassandra", "users")
{% endhighlight %}

In a typical query scenario we couldn’t query by age because it’s not a part of a partition or clustering key. We could query and explicitly allow filtering but this is a huge performance impact and it is not recommended. On the other hand, we could use secondary indexing but this is also at the cost of performance since cardinality is high. We can solve this problem using Spark. As an example, we could execute transformation and get count

{% highlight scala %}
val adults = usersTable.filter(_.age > 21)
val total = adults.count
{% endhighlight %}

This is great but since we are talking about SparkSQL we can also query using SQL statements against the Cassandra table like this

{% highlight scala %}
case class User(username: String, name: String, email: String, age: Int)
sc.cassandraTable[User]("testsparkcassandra", "users").registerAsTable("users")
var adults = sql("SELECT * FROM users WHERE age > 21")
{% endhighlight %}

Now when we execute count the actual query will be executed and we have used SQL syntax to execute query which could not be possible on Cassandra. Keep in mind that it is always recommended to execute filtering on the server side so that the least required amount of data is being transferred over network. Selecting only required columns or using correct where clauses does this job pretty well.

Spark has really helped in enabling wider audiences of people to execute data analysis and also raised the performance bar really high. In combination with Cassandra, our favorite database, it really makes a great stack for data streaming, processing and analysis. Being distributed helps with scaling and project documentation is really detailed. This is the reason why we put this combo in front and why we can cover a high percentage of use cases with it.

[spark-cassandra-connector-github]: https://github.com/datastax/spark-cassandra-connector
