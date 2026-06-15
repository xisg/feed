---
title: Some fun with Redis Cluster testing
url: http://antirez.com/news/69
published: "2013-12-18T15:32:21Z"
feed: antirez
guid: http://antirez.com/news/69
---

# Some fun with Redis Cluster testing

One of the steps to reach the goal of providing a "testable" Redis Cluster experience to users within a few weeks, is some serious testing that goes over the usual "I'm running 3 nodes in my macbook, it works". Finally this is possible, since Redis Cluster entered into the "refinements" stage, and most of the system design and implementation is in its final form already.

In order to perform some testing I assembled an environment like that:

\\* Hardware: 6 real computers: 2 macbook pro, 2 macbook air, 1 Linux desktop, 1 Linux tiny laptop called EEEpc running with a single core at 800Mhz.

\\* Network: the six nodes were wired to the same network in different ways. Two nodes connected via ethernet, and four over wifi, with different access points. Basically there were three groups. The computers connected with the ethernet had 0.3 milliseconds RTT, other two computers connected with a near access point were at around 5 milliseconds, and another group of two with another access point were not very reliable, sometime some packet went lost, latency spikes at 300-1000 milliseconds.

During the simulation every computer ran Partitions.tcl (http://github.com/antirez/partitions) in order to simulate network partitions three times per minute, lasting an average of 10 seconds. Redis Cluster was configured to detect a failure after 500 milliseconds, so these settings are able to trigger a number of failover procedures.

Every computer ran the following:

Computer 1 and 2: Redis cluster node + Partitions.tcl + 1 client

Computer 3 to 6: Redis cluster node + Partitions.tcl

The cluster was configured to have three masters and three slaves in total.

As client software I ran the cluster consistency test that is shipped with redis-rb-cluster (http://github.com/antirez/redis-rb-cluster), that performs atomic counters increments remembering the value client side, to detect both lost writes and non acknowledged writes that were actually accepted by the cluster.

I left the simulation running for about 24 hours, however there were moments where the cluster was completely down due to too many nodes being down.

The bugs

===

The first thing that happened in the simulation was, a big number of crashes of nodes… the simulation was able to trigger bugs that I did not noticed in the past. Also there were obvious mis-behavior due to the fact that one node, the eeepc one, was running a Redis server compiled with a 32 bit target. So in the first part of the simulation I just fixed bugs:

7a666ac Cluster: set n->slaves to NULL in clusterNodeResetSlaves().

fda91db Cluster: check link is valid before sending UPDATE.

f57bb36 Cluster: initialize todo\_before\_sleep flags to 0.

c70c0c6 Cluster: use proper type mstime\_t for ping delay var.

7c1cbdc Cluster: use an hardcoded 60 sec timeout in redis-trib connections.

47815d3 Fixed clearNodeFailureIfNeeded() time type to mstime\_t.

e88e6a6 Cluster: use long long for timestamps in clusterGenNodesDescription().

The above commits made yesterday are a mix of bugs reported by valgrind (for some part of the simulation there were nodes running over valgrind), crashes, and misbehavior of the 32 bit instance.

After all the above fixes I left the simulation running for many hours without being able to trigger any crash. Basically the simulation “payed itself” just for this bug fixing activity… more minor bugs were found during the simulation that I’ve yet to fix.

Behavior under partition

===

One of the important goals of this simulation was to test how Redis Cluster performed under partitions. While Redis Cluster does not feature strong consistency, it is designed in order to minimize write loss under some very common failure modes, and to contain data loss within a given max window under other failure modes.

To understand how it works and the failure modes is simple because the way Redis Cluster itself works is simple to understand and predict. The system is just composed of different master instances handling a subset of keys each. Every master has from 0 to N replicas. In the specific case of the simulation every master had just one replica.

The most important aspect regarding safety and consistency of data is the failover procedure, that is executed as follows:

\\* A master must be detected as failing, according to the configured “node timeout”. I used 500 milliseconds as node timeout. However a single node cannot start a failover if it just detects a master is down. It must receive acknowledgements from the majority of the master nodes in order to flag the node as failing.

\\* Every slave that flagged a node as failing will try to be elected to perform the failover. Here we use the Raft protocol election step, so that only a single slave will be able to get elected for a given epoch. The epoch will be used in order to version the new configuration of the cluster for the set of slots served by the old master.

Once a slave performs the failover it reclaims the slots served by its master, and propagates the information ASAP. Other nodes that have an old configuration are updated by the cluster at a latter time if they were not reachable when the failover happened.

Since the replication is asynchronous, and when a master fails we pick a slave that may not have all the master data, there are obvious failure modes where writes are lost, however Redis Cluster try to do things in order to avoid situations where, for example, a client is writing forever to a master that is partitioned away and was already failed over in the majority side.

So this are the main precautions used by Redis Cluster to limit lost writes:

1) Not every slave is a candidate for election. If a slaves detects its data is too old, it will not try to get elected. This in practical terms means that the cluster does not recover in the case where none of the slaves of a master are able to talk with the master for a long time, the master fails, the slave are available but have very stale data.

2) If a master is isolated in the minority side of the cluster, that means, it senses the majority of the other masters are not reachable, it stops accepting writes.

There are still things to improve in the heuristics Redis Cluster uses to limit data loss, for example currently it does not use the replication offset in order to give an advantage to the slave with the most fresh version of data, but only the “disconnection time” from the master. This will be implemented in the next days.

However the point was to test how these mechanisms worked in the practice, and also to have a way to measure if further improvements will lead to less data loss.

So this is the results obtained in this first test:

\\* 1000 failovers

\\* 8.5 million writes performed by each client

\\* The system lost 2000 writes.

\\* The system retained 800 not acknowledged writes.

The amount of lost writes could appears to be extremely low considered the number of failovers performed. However note that the test program ran by the client was conceived to write to different keys so it was very easy when partitioned into a minority of masters for the client to hit an hash slot not served by the reachable masters. This resulted into waiting for the timeout to occur before of the next write. However writing to multiple keys is actually the most common case of real clients.

Impressions

===

Running this test was pretty interesting from the point of view of the paradigm shift from Redis to Redis Cluster.

When I started the test there were the bugs mentioned above still to fix, so instances crashed from time to time, still the client was almost always able to write (unless there was a partition that resulted into the cluster not being available). This is an obvious result of running a cluster, but as I’m used to see a different availability patter with what is currently the norm with Redis, this was pretty interesting to touch first hand.

Another positive result was that the system worked as expected in many ways, for example the nodes always agreed about the configuration when the partitions healed, there was never a time in which I saw no partitions and the client not able to reach all the hash slots and reporting errors.

The failure detection appeared to be robust enough, especially the computer connected with a very high latency network, from time to time appeared to be down to a single node, but that was not enough to get the agreement from the other nodes, avoiding a number of useless failover procedures.

At the same time I noticed a number of little issues that must be fixed. For example at some point there was a power outage and the router rebooted, causing many nodes to change address. There is a built-in feature in Redis Cluster so that the cluster reconfigures itself automatically with the new addresses as long as there is a node that did not changed address, assuming every node can reach it.

This system worked only half-way, and I noticed that indeed the implementation was not yet complete.

Future work

===

This is just an initial test, and this and other tests will require to be the norm in the process of testing Redis Cluster.

The first step will be to create a Redis Cluster testing environment that will be shipped with the system and that the user can run, so it is possible that the cluster will be able to support a feature to simulate partitions easily.

Another thing that is worthwhile with the current test setup using partitions.tcl is the ability of the test client, and of Partitions.tcl itself, to log events. For example with a log of partitions and data loss events that has accurate enough timestamps it is possible to correlate data loss events with partitions setups.

If you are interested in playing with Redis Cluster you can follow the Redis Cluster tutorial here: http://redis.io/topics/cluster-tutorial
[Comments](http://antirez.com/news/69)
