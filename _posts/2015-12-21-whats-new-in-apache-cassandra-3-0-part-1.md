---
title:  "What's new in Apache Cassandra 3.0 - part 1"
date:   2015-12-21 18:22:56 +0200
---
![an image alt text]({{ site.baseurl }}/images/2000px-cassandra_logosvg.png "What's new in Apache Cassandra 3.0")

#### **A bit of a background**

In the world of a fast growing number of NoSql databases and fast, scalable and distributed systems, we are solving problems that we couldn’t solve with previously proven technologies due to changing requirements. We are indeed solving some of the problems of a single instance database but we have been introducing numerous new problems related to distributed systems in general and problems that come with different physical data models of new databases. One of the examples of this change is definitely the Cassandra database with its partitioned row store. It’s fast, it’s scalable and it’s completely distributed and with these attributes it does solve a lot of problems. On the other hand it’s completely different from any relational database we used before.

Data modeling in Cassandra requires us to manually denormalize data into tables based on how we are going to read the data. This is called a query based data model. The main problem lies in how we are used to query data in relational databases: you define a data model based on the logical model and you can query it any way you need. Denormalized data in Cassandra requires you to save the data as it’s going to be queried. After understanding how the data is stored in partitions you’ll quickly encounter the problem of querying data without specifying the partition key. The only way to solve this is by using secondary indexes, which is not recommended, especially as they are a bad fit for high cardinality data. This means that, by executing query, all nodes in the cluster will get hit and perform query, which adds stress to the cluster and can pile up latency on the request. This is solved by adding an another table with saved data so that it can serve the query. As the number of tables per entity grows, we need to add more insert statements into our logged batch and have more code in the application. In Cassandra 3.0 there is a new feature called Materialized Views which handles this automatically, which removes the need for additional application code and leverages the existing Cassandra read and write paths.

So, for example, let’s create a user table for our application that defines the user entity

{% highlight sql %}
CREATE TABLE user (
    username text,
    password text,
    email text,
    company text,
    PRIMARY KEY (username)
);
{% endhighlight %}

Now, if we are authenticating the user by a third party service we are going to have to query our user by its email. This is a typical problem and it’s easily solvable in a relational database by adding another query on the user table by equality comparer in the email field. In Cassandra, this can be done using secondary indexes but the query would need to hit all nodes in order to return the result and it would create a possible performance issue since it’s a case of high cardinality data. The second option is to create a second table with all the user information as in the [user] table but partitioned by email.

{% highlight sql %}
CREATE TABLE user_by_email (
    email text,
    username text,
    password text,
    company text,
    PRIMARY KEY (email)
);
{% endhighlight %}

It would require us to write additional code in the application to handle CRUD and everything would have to be executed in logged batches. Materialized views to the rescue: instead of creating the [user_by_email] table and adding application code we can write a materialized view and just query it when necessary.

{% highlight sql %}
CREATE MATERIALIZED VIEW user_by_email AS
SELECT * FROM user
WHERE email IS NOT NULL
PRIMARY KEY (email, username);
{% endhighlight %}

Note that this materialized view is created by selecting * from the user table thus any field change in user table will get reflected in this materialized view. This sounds like a good solution and table data is synchronized by the database.

Another case that I would like to note is creating a table to support query by the company.

{% highlight sql %}
CREATE TABLE user_by_company (
    company text,
    username text,
    email text,
    PRIMARY KEY (company)
);
{% endhighlight %}

In this case, creating a secondary index in the company field in the user table could be a solution because it has much lower cardinality than the user's email but let’s solve it with performance in mind. Secondary indexes are always slower than dedicated table approach. This can also be solved by using materialized views like this:

{% highlight sql %}
CREATE MATERIALIZED VIEW user_by_company AS
SELECT company, username, email FROM user
WHERE company IS NOT NULL AND email IS NOT NULL
PRIMARY KEY (company, username);
{% endhighlight %}

The difference here is that the table containing users by company naturally should not contain the user's password so the select statement for creating this view has to define the fields from the [user] table. In case of adding or deleting fields in the [user] table this select statement would have to be updated to reflect changes.

So, how good are materialized views and do they solve all our development problems with multiple tables per entity? To some extent, yes. Let’s talk about the good and the bad.

#### **The good stuff**

It's a great way to easily add functionality and additional queries to your app. There is no need for any data migration when creating a materialized view compared to a manually created table because historic data is automatically pulled into the view on creation. Any data altering is automatically reflected in the view and the fields can be defined in the creating statement. The view table is deleted when the source table is deleted. We can combine user defined functions or UDFs with materialized views to get even more refined query results.

#### **The bad stuff**

Mutations to materialized views are executed as a logged batch and are sent out to the replicas while waiting for the consistency requirements to be fulfilled. This process is a bit complicated and it’s described in this article by [Datastax on materialized views][datastax-materialized-views] in Cassandra 3.0. Performing delete on the source table in many cases produces more than one tombstone on materialized views which can be a problem with big delete operations (remember the tombstone thresholds). If a source table is updated in such a way that we will get a new row in the view table, that means that we have to tombstone the previous row. One of the main problems is that materialized views require read before write which is a killer in Cassandra practices because it requires additional consistency checks and presents performance hit that impacts writes. There is also a low cardinality problem we have to account for. If we have a lot of users working for a single company, the user_by_company view will generate a hot spot.

#### **The verdict**

In the previous years, I worked a lot with multiple tables per entity and it always worked out great. I can control which statements are executed in batches either logged or unlogged and there is more control over consistency requirements. Materialized views are a great add-on for Cassandra but taking into account the complexity of insert/update procedures and the number of tombstones that are created I cannot agree that this is a great solution for all such cases. I would even go that far to say that materialized views are not that great as they seem. On the other hand, they do provide an easy way to expand your queries and there is no need for additional tools to insert historic data into your new manually created table. We solved most of the issues of database schema updates with our [cassandra migration tool][cassandra-migration-tool-github] and with that in mind I would think twice before embracing materialized views easily.

It’s up to you to figure out what suits you best.

[cassandra-migration-tool-github]: https://github.com/smartcat-labs/cassandra-migration-tool-java
[datastax-materialized-views]: http://www.datastax.com/dev/blog/new-in-cassandra-3-0-materialized-views
