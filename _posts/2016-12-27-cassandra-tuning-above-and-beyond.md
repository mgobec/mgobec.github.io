---
title:  "Cassandra Tuning - Above and Beyond"
date:   2016-12-27 11:27:14 +0200
---

![an image alt text]({{ site.baseurl }}/images/2000px-cassandra_logosvg.png "Cassandra Tuning - Above and Beyond")

This September I held a presentation on Cassandra Summit 2016 about tuning Apache Cassandra for low latency while deployed on AWS.

This is the abstract of the presentation:

*There are many aspects of tuning Cassandra for production and a lot can go wrong: network splits and latency, hardware issues and failure, data corruption, etc. Most are mitigated with Cassandra's architecture but there are use cases where we need to dig deep and tune all layers to get the result we need to achieve specific business goals.
We will explore such case where we had to tune Cassandra for performance but also have consistent results on 99.999% of the queries. Getting even to 99 percent was relatively easy, but pushing those extra nines involved a lot of work. There are many nuts and bolts to turn and tune in order to get consistent results.
We will cover biggest latency-inducing factors and see how to set up metrics and tackle inevitable issues when doing cloud-based deployments. We will get into one of the major "sins" regarding AWS deployment by demystifying EBS based storage and talk about how we can leverage OS properties while tuning for high read performance.*

Here is a link to the video:

<p><iframe width="560" height="349" src="https://www.youtube.com/embed/bQRjfHwjAL4?feature=oembed" frameborder="0" allowfullscreen=""></iframe></p>

and here are the [SLIDES][cassandra-summit-2016-slides].

[cassandra-summit-2016-slides]: http://www.slideshare.net/DataStax/cassandra-tuning-above-and-beyond-matija-gobec-smartcat-cassandra-summit-2016
