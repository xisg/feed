---
title: Pat's Big Deal, and Transaction Coordination
url: http://brooker.co.za/blog/2024/01/23/big-deal.html
published: "2024-01-23T00:00:00Z"
feed: brooker
guid: http://brooker.co.za/blog/2024/01/23/big-deal
---

# Pat's Big Deal, and Transaction Coordination

Working together towards a common goal.

I have a lot of opinions about Pat Helland’s CIDR’24 paper [Scalable OLTP in the Cloud: What’s the BIG DEAL?](https://www.cidrdb.org/cidr2024/papers/p63-helland.pdf) [1](#foot1). Most importantly, I like the BIG DEAL that he proposes:

> Scalable apps don’t concurrently update the same key.

> Scalable DBs don’t coordinate across disjoint TXs updating different keys.

In exchange for fulfilling their sides of this big deal[2](#foot2) the application gets a database that can scale[6](#foot6), and the database gets a task to do that allows it to scale. The cost of this scalability, for the application developer, is having to deal with a weird concurrency phenomenon called _write skew_. In this post, we’ll look at _write skew_, why not preventing it helps databases scale, and how we can alter Pat’s big deal to prevent _write skew_ and get _serializability_.

In particular, we’ll try answer two big questions:

- Is Pat’s big deal the best deal available?
- Would a serializable big deal be better for application programmers?

**Snapshot Isolation**

The big deal experiences _write skew_ because of the database isolation level that Pat chose: snapshot isolation. Other than this particular weird behavior, the application can pretend that concurrent transactions run in a serial total order, which makes application developer’s lives easy. Nobody likes reasoning about concurrency, and reasoning about concurrency while correctly implementing business logic is pretty hard, so that’s comforting.

There are a lot of ways to talk about these concurrency anomalies in the database literature. Tens, at least. The one I think is most accessible to developers is Martin Kleppmann’s [Hermitage](https://github.com/ept/hermitage/tree/master), a set of minimal tests that illustrate each of the weird things that databases users can experience.

The test setup is super simple:

```
create table test (id int primary key, value int);
insert into test (id, value) values (1, 10), (2, 20);

```

Next, we make two connections to the database we’ll call _A_ and _B_, and run a transaction through each connection. For each row of the table, imagine us running that statement, waiting for it to end, and then going on to the next row.

![SQL for Martin Kleppmann's G2-item example from Hermitage](/blog/images/write_skew.png)

In a _snapshot isolated_ database, both _A_ and _B_ commit. In a _serializable_ database, one of them needs to fail: there’s no way to order these two transactions into a serial order that makes sense (either _A_ needs to see _B_’s writes, or _B_ needs to see _A_’s)[3](#foot3).

This isn’t some strange academic edge case. For example, consider an application that reads from the _stuff in warehouse_ table, then writes to an _order_ table (without updating the warehouse, because the stuff is still there). The snapshot isolated version will sell some things too many times.

The big deal then comes with a real trade-off, and forces the application programmer to take some care to ensure correct results (but, importantly, less so than at lower levels like _read committed_).

**Is SI Ideal for the Big Deal?**

To understand whether SI is ideal for the big deal, we need to look at another angle on isolation: what coordination the database needs to do to achieve that level[4](#foot4). Let’s say we have a transaction _A_, and that transaction starts at some time $\\tau^A\_s$ and commits at some time $\\tau^A\_c$. To offer transaction _A_ snapshot isolation, we need to offer it two properties:

Promise $1$: _A’s_ reads see all the effects of transactions that committed before _A_ started (i.e. before $\\tau^A\_s$), and none of the effects of transactions that committed after.

Promise $2\_{si}$: _A_ can only commit if none of the keys it _writes_ have been written by other transactions between _A_ starts and when it commits (i.e. between $\\tau^A\_s$ and $\\tau^A\_c$).

There are many ways to implement these guarantees, but the implementation decisions aren’t particularly important here. What’s important is the coordination needed. There doesn’t appear to be any inherent reason that promise 1 (the read guarantee) requires any coordination at all. For example, _A_ could be given its own read replica which is completely disconnected from the stream of updates for the duration. It’s the second step where coordination is required: either to block the other writers (write locks), prevent other from committing, or detect the writes at the time _A_ comes to commit. All of those require coordinating with other writers, either continuously through the transaction or at commit time.

Now, let’s consider what the second promise would look like if we wanted to offer _serializability_ to _A_ (and therefore prevent that write skew anomaly we talked about earlier).

Promise $2\_{ser}$: _A_ can only commit if none of the keys it _read_ have been written by other transactions between _A_ starts and when it commits (i.e. between $\\tau^A\_s$ and $\\tau^A\_c$).

We’ve changed one word in the definition, but entered something of a rabbit hole. The snapshot version of Promise 2 only needs to coordinate on writes, and find write-write conflicts between transactions. It only needs to keep track of keys that are written, and talk to the machines that are responsible for detecting conflicts on those keys.

The serializable version, on the other hand, needs to track all the keys _A_ read (and the keys _A_’s predicates could have read but didn’t see), and then look for writes from other transactions to those keys. This doesn’t seem that different, really, but is a practical problem because it’s very easy (and common) for applications to make those read sets very big. For example, imagine _A_ does:

```
SELECT id FROM stock WHERE type = 'chair' ORDER BY num_in_stock DESC LIMIT 1;

```

In the serializable version of the promise, now _every chair_ is in _A_’s read set. _A_ will then need to conflict with any other transaction that writes to any chair row, even if it’s not the one that _A_ picked[7](#foot7). If we sold just one chair of any type during the time _A_ is running, the serializable version of _A_ couldn’t commit. What’s worse, from the Big Deal’s perspective of thinking about scalability, is that _A_ would need to coordinate with the machines that own _all_ chairs. In a distributed database, that’s a lot of coordination!

In the snapshot version, _A_ would only need to coordinate with the machines that own any chairs it touched. Like this:

```
UPDATE stock SET num_in_stock = num_in_stock - 1 WHERE id = 'the cool chair the customer chose';

```

The snapshot version of _A_ would only need to coordinate with that one machine that owns that one critical chair. Changing that one word between Promise $2\_{si}$ and Promise $2\_{ser}$ significantly changed the required coordination patterns.

But does that change the asymptotic scalability of the database? It does in this example (because of $O(\\textrm{chairs})$ coordination for serializability and $O(1)$ for snapshot). But does it in general? Only if we believe that applications’ read behavior, in general, is asymptotically different from their write behavior (otherwise we’re just moving constants around[5](#foot5)). Specifically, that the number of _read-write_ edges is asymptotically different from the number of _write-write_ edges in the data access graph.

This is the sense in which we can say that snapshot isolation is better for Pat’s Big Deal: making an assumption that applications access data in an asymptotically different way from how they write it.

**I Am Altering the Deal**

Wow, ok, that’s a lot of writing. But now I think we can propose, in the tradition of Roosevelt, a New Deal. A serializable deal. Without write skew. First, let’s remind ourselves about Pat’s Big Deal:

> Scalable apps don’t concurrently update the same key.

> Scalable DBs don’t coordinate across disjoint TXs updating different keys.

And now, our serializable New Big Deal.

> Scalable apps don’t update keys that are frequently read by other concurrent processes, and try not to read keys that are frequently written.

As the kids say: Oof.

> Scalable DBs don’t coordinate across disjoint TXs.

Well, we’ve simplified that one, but have made the definition of _disjoint_ much more complex by defining it in terms of both reads and writes.

**Pray I Don’t Alter it Any Further**

The serializable version of the Big Deal is simpler for the application programmer from a correctness perspective. In fact, they can basically assume that concurrency doesn’t exist, which is very nice indeed. But it’s harder on the application programmer from a scalability and performance perspective, in that they have to be much more careful about the reads they do to get good scalability. It’s clear that’s not an easy win, but might be a net win in some circumstances.

**Footnotes**

1. Also check out Murat Demirbas’s [analysis of the paper](http://muratbuffalo.blogspot.com/2024/01/scalable-oltp-in-cloud-whats-big-deal.html).
2. A big deal and a big deal. A major agreement, and very important.
3. [Hermitage](https://github.com/ept/hermitage/tree/master) separates this phenomenon into _write skew_ (G2-item) and _anti-dependency cycles_ (G2). They’re closely related, with the latter focusing on predicate reads.
4. We’re skimming over some deep water here to make a point - isolation implementation is the topic of decades of database research, and decades of attempts to formalize (e.g. [Adya](https://pmg.csail.mit.edu/papers/adya-phd.pdf), [Crooks, et al](https://www.cs.cornell.edu/lorenzo/papers/Crooks17Seeing.pdf)), implement (e.g. [Gray and Reuter](https://www.amazon.com/Transaction-Processing-Concepts-Techniques-Management/dp/1558601902/), [Kung and Robinson](https://www.eecs.harvard.edu/~htk/publication/1981-tods-kung-robinson.pdf)), and explain isolation levels. Please forgive me some simplification.
5. But let’s be clear - in the actual practical world moving constants around is super important.
6. In this post, I’m using the word _scale_ (and related words like _scalable_ and _scalability_) in the asymptotic sense Pat uses in his paper. Note that this is different from the sense that most folks use them in.
7. Assuming _A_ does any writes. If _A_ is read-only, it can “commit” at $\\tau^A\_c = \\tau^A\_s$ (which, because of our Promise 1, is always a valid and serializable thing to do).
