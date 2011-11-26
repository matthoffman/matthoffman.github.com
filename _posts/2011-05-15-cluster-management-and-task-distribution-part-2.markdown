--- 
layout: post
title: "Cluster Management and Task Distribution part 2: Caching"
summary: "where I evaluate a third option in round-about fashion"
tags: 
- Big Data
- Programming
---

<h2>The question</h2>
Which technology stack makes more sense for a distributed caching framework?

<h2>The backstory</h2>

I've gotten a fair amount of hits on <a href="http://matt-thinks-so.com/2011/01/29/cluster-management-and-task-distribution-zookeeper-vs-jgroups.html">this post</a> comparing JGroups and Zookeeper for task distribution and cluster management. 
I wanted to write an update of what I've investigated so far... unfortunately, my free time to investigate this specific issue has gone kapoof. Getting rid of the single point of failure in task distribution has gone on the backlog again. 

However, in the meantime, I've done some investigation on distributed caches. This was the same app, but a different need --- I needed a quick, practical way to distributed data across the cluster, so I tested Infinispan and Hazelcast. <br>

Infinispan uses JGroups behind the scenes to do its clustering; Hazelcast has its own homegrown set of protocols.

Zookeeper, for all its pluses as a cluster management option, doesn't have anything to offer for this use case. And since distributed caching came up first as a production requirement, Zookeeper may lose out by default; either of these caches can also be adapted for cluster management and task distribution (in the form of a fully-replicated cache, if nothing else). So if they're already in the stack, and using them isn't too much work, that's likely what we'll use. It frustrates me to have to make technical decisions this way, but the way that our timelines go, that's often the way it is. 

<h2>Why not Terracotta?</h2>

Terracotta is often at the top of the list when talking about distributed caches, so why wasn't it here? 
Three reasons: 
<ul>
  <li>This particular client has had bad experiences with Terracotta in the past, and was resistent to installing it here.</li>
  <li>Terracotta requires a server component, which is more operational overhead.  It also makes it more difficult to transparently switch between clustering technologies.</li>
  <li>Terracotta's license (at least, last I checked) is sort-of free, but with a stronger-than-normal attribution clause; adding commercial license cost wasn't possible for this minor release. The attribution clause wasn't entirely a deal-breaker, but it's certainly less appealing.</li>
</ul>

The first reason there is more significant than the other two. Terracotta may still end up as a distributed caching option in the future.

Unfortunately, this isn't as in-depth or thorough of an investigation as <a href="http://matt-thinks-so.com/2011/01/29/cluster-management-and-task-distribution-zookeeper-vs-jgroups.html">part 1</a>. I'm not going to go back through the requirements and then see how the candidates stack up. Infinispan and Hazelcast, our two main candidates, have a very similar feature list on paper, so it came down to a hands-on investigation instead. 

<h2>The plan</h2>

I already had a generic cache interface that the app was using, so my goal was to write an implementation for both Infinispan and Hazelcast and allow us to switch between the two at startup time. That proved to be a bit of a trick --- both have Java configuration options (instead of XML-only configuration) in their most recent versions. Both have some tricks to them.<br>

My goal was to configure either cache with just two properties: 
<pre>
cache.type=hazelcast
cache.members=192.168.221.10, 192.168.221.11, 192.168.221.12, 192.168.221.13
</pre>

The first obviously tells the app whether to use Hazelcast or Infinispan. The second points to at least some of the other cache members. That's necessary because multicast --- which is the default discovery option for both of these products --- isn't available on EC2, which is one of our possible deployment options. For Hazelcast, the list of nodes doesn't need to be a complete list; it will use the listed nodes as seeds for a complete list. Infinispan, on the other hand, is more unclear.

<h2>Implementation</h2>

I ran into a few issues. For example, in Infinispan, my clever code that parsed the address list, found if one of them applied to the current server, and set the bind address appropriately ran into some snags.  Netstat showed the following: 

<pre>
tcp        0      0 ::ffff:10.192.171.127:8016  :::*                        LISTEN      
</pre>

It was binding an IPv6 compatibility address. And my "separate port from address" code isn't smart enough to handle IPv6 addresses, so this wasn't going to work.
I worked around it (in this case, I hardcoded out IPv6 addresses...that's not on my roadmap right now) and eventually got both working.

Writing the code that actually used the distributed caches was painless. The implementations of my simple cache interface came together without any problem; both implement both ConcurrentMap and JCache's Cache interface, so they both fit in easily.

Unfortunately, I was still running into <a href="https://issues.jboss.org/browse/JGRP-1253">Infinispan issues</a> when a node died and then later rejoined the cluster. 

Hazelcast, on the other hand, worked out of the box in this case, so that's what is going on into testing at the client site.

All in all, Hazelcast surprised me.  It seems to be gaining acceptance on the web:  anecdotally, posts on Stack Overflow from two years ago tended to say "What's Hazelcast?".  Last year they would say "Hazelcast? That's unproven". This year, when it's showing up on recommendation lists. Granted: small sample size. But that's what we have to go on.
In my testing, it works out of the box and does what it's supposed to.

Now, it's worth noting that adding Hazelcast -- or any distributed caching -- doesn't come without a cost. Unfortunately, I don't have that cost quantified, although I'll try to update this post with measurements if people are curious. But the time spent writing data to and reading data from distributed cache often slow down the application to the point where two machines with a distributed cache are slower than one machine without[^1]. The break-even point depends on the technology, the workload, and the hardware, and how careful you are with what is written to and read from the distributed cache. The higher the consistency and durability guarantees of the cache, the more painful that cost will be.  

It's probably worth digging into this cost briefly. It might seem at first glance that this shouldn't be the case: after all, you're just storing data in memory, and when a machine on the cluster needs some data, it asks other machines before going to the database. As long as getting the data from another local machine is faster than going to the database and the cache hit rate is reasonably high, it seems like it should be faster.  Alas, it is not the case. 

It doesn't work that way for a couple of reasons: first, because you don't have any durability guarantees if, after calling cache.put(), it just lives in that first machine's local memory. If that machine goes down, the data is lost. So most caches (including Infinispan and Hazelcast) will, by default, block the put() call until that data has also been written out to at least one other machine. Of course, if you don't need that durability, you have options -- you can turn off replication entirely, or let it happen in a write-behind queue, so that there's only a limited window of time in which the data lives on only one machine.

The cache is also likely to be slower because attempts to be clever about which machine holds each element in the cache. It tries to distribute cached data around the available nodes, as well as offer a quick way to compute which machine owns each piece of data, so that it doesn't have to broadcast a "who has cache key X?" every time you do a cache.get(X).  By default, it does this using a hash function on the cache key.  That means that, when you call cache.put(..) on machine A, there's no guarantee that the data will stay on machine A. So after calling put(), you may have to wait for the data is written to the machine that is declared the owner of that value, based on the hash of the cache key. It's worth noting that Terracotta doesn't work this way -- data ownership is a function of the node that last used the data. But Infinispan and Hazelcast both do. 

Now, those who are used to distributed systems, this will be old hat, but it surprises a lot of people.  Two machines is very often slower than one. That's why folks moving to Hadoop and HBase or other dedicated clustering datastores designed to scale to dozens o hundreds of machines find that they don't start equaling single-machine performance until they move to several machines. Anecdotally, that is often around 5 machines, although it depends heavily on the workload -- how much data is being written to and read from distributed storage vs. other uses of time (CPU, other I/O). 

So, the moral of the story: be very careful what's going in and out of cache. "Transparent caching" cannot be -- it has a very significant cost over and above the cost of just writing the blocks of local memory, UNLESS you remove enough consistency guarantees that it can happen less obtrusively after the fact.

  [^1]:  If you haven't seen it, check out the ["numbers every computer engineer should know"](http://www.quora.com/What-are-the-numbers-that-every-computer-engineer-should-know-according-to-Jeff-Dean) from Jeff Dean's talk "Building Software Systems at Google and Lessons Learned" at Stanford ([slides](http://www.cs.cornell.edu/projects/ladis2009/talks/dean-keynote-ladis2009.pdf), [video](http://goo.gl/0MznW])).  It has some really useful order-of-magnitude values you can use for back-of-the-envelope estimates of the cost of distributed caching.

In related news, Akka 2.0 will gain the clustering support that's currently reserved for the commercial offering ("Cloudy Akka")... I'll be interested to see what that looks like. 

<h2>Conclusion</h2>

I've split the conclusion into a separate post, [here](/2011/11/05/cluster-management-and-task-distribution-part-3.html).

