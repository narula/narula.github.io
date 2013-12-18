---
layout: post
title:  "DRAFT -- The Intricacies of Consistency"
date:   2013-12-15 13:48:14
categories:
comments: true
---

# What is this about?

We spend a lot of time discussing consistency trade-offs without
discussing what consistency really _is_ in different contexts, and why
it's even a good idea to begin with.

There are three primary contexts that often get conflated when we
discuss consistency: concurrent processing, distributed consistency,
and traditional database consistency and isolation.

# In the beginning, there was a program

Let's discuss concurrent programming first.  Imagine the the simplest
case -- one single-threaded process running on one single core
machine, reading and writing to memory.

Here we get consistency for free; all of the process's reads and
writes are ordered because it is single-threaded, and only one memory
operation is happening at a time.  In fact only one thing is *ever*
happening in the system at a time.  The programmer can assume that all
of the program's instructions execute as though they are the only
things running on this machine, because they are.  We call this
_isolation_.  We don't need fancy primitives; life is good.

# Consistent and Transactional

This simple example falls apart as soon as we consider a machine with
multiple cores.  By forcing our program to run in a single thread, we
are leaving half the CPU performance on the table.  So we spawn a copy
-- two threads, excellent!  This one little action has made our lives
extremely difficult.  Now we have to reason about all possible
interleavings of two instances of this program, and consider what
could go wrong.  Oh and the more cores the more instances we need, and
the more interleavings to reason about.

There are two main problems here: One is that on many cores we have
multiple readers and writers, and in fact data is replicated on
different L1, L2, and L3 caches.  We want to make sure each program
appears _sequentially consistent_ -- as though all operations were
executed in some sequential order, and that this global sequential
order is consistent with each thread's individual order.  As an
example, there should be some order where if Process 1 reads A=2 and
Process 2 reads A without any intervening writes to A, it should read
A=2 as well.  Fortunately, on a multi-core processor, the cache
coherence protocol does most of this for us.

Another problem we have is that we now might want to do multiple reads
and writes atomically without anyone interrupting and changing state
out from under us; for example we might want to read and write a
memory location to decrement a variable.  We want these sets of
instructions to execute _transactionally_, the way they would if a
given program were the only thing executing on the machine, and we
want to either see the result of all the instructions or none of them,
never partial state. We can use synchronization primitives for the
first task and something like hardware transactional memory for the
second.

When we extend these goals to a distributed setting (clients issuing
reads and writes from different machines) life gets harder.  When we
extend them to a system where we have _datastores_ running on
different machines and introduce independent failures, life gets so
hard we begin to wonder if we should have just gone to medical school
and been a doctor like our parents wanted instead of studying computer
science.

But we will prevail!  Those who came before us not only invented
techniques to manage the chaos, but were able to describe the
semantics of what we could get out of our systems under certain
circumstances, so we can distance ourselves from hideous
implementations and corner cases of failures to program with those
blessed things, abstractions and guarantees.  

# Consistency Levels

Abstractions and guarantees -- the purpose of consistency is not to
maintain some inefficient, platonic ideal of what our data should be,
but to _make our lives easier_.  It is extremely difficult to reason
about systems that don't provide implementation-independent
guarantees!  Can you imagine a world where you had to read and
understand the entire code base of each datastore you used, just to
get some idea of how it would read and write your data?

One thing I think people gloss over here is how isolation works with
consistency.  The first models I'm going to describe only involve
ordering between single-key reads and writes.  They say nothing about
multi-key operations, which are also important.

## Eventual Consistency, or Really We Should Just Call This Nothing

Eventually consistent systems provide one guarantee and one guarantee
only -- eventually, if stuff stops happening, every process involved
will agree on the same value in the same place.  Up until then,
process 1 can return 42 every single time and process 2 can return "I
can see your underwear" and this is totally fine and allowed (and when
do things ever just "stop"?).

I do not have much to say about eventual consistency.  It works ok for
a some people, I guess.  I think a lot of developers actually go to
great lengths in their applications to fix up the anomalies that can
happen because of eventual consistency.  Maybe they need to do this
anyway, cause bugs and bit flips and whatnot.  Who am I to say.

## Causal Consistency, or Have Fun Reasoning About This

Ok so the idea here sounds great -- you're looking at a page on
Facebook, and you see a friend's post in your newsfeed.  Then you
click over to your friend's profile page and nada.  You went to a
different replica or something and it hasn't made it over there yet
(totally legal under EC).  Causal consistency to the rescue!
Basically the idea is that you always see what you've already seen and
things that went into things you've already seen.  This is 1) only
within the system, so if you call your friend or read from another
datasource he might have stuff you haven't seen yet 2) requires
keeping a bunch of state around and 3) I don't think it's even that
easy to reason about.  Either the system measures _too much_
causality, or the developer is required to reason about and tell
the system what reads should satisfy causal relationships and what
shouldn't.

## Sequential Consistency

This is the guarantee you get when you are running your program
(possibly with multiple threads) on one computer.  Replicating this in
a distributed setting requires coordination, but you can pretend like
you're programming on one machine, which is really nice.  Here's the
definition:

> Sequential consistency ...was first defined as the property that
> requires that "... the result of any execution is the same as if the
> operations of all the processors were executed in some sequential
> order, and the operations of each individual processor appear in
> this sequence in the order specified by its program.".

Taken from Wikipedia, who stole it from Leslie Lamport.

This says that "the operations of each individual processor appear in
this sequence in the order specified by its program".  However don't
think all is well.  Cody pointed out this example, which made me
realize sequential consistency is actually NOT so easy to understand.
In the following pictures time moves to the right, and each P_i is a
different processor.  Which of these is sequentially consistent?
Both?  Neither?

![Sequential example]({{ site.url }}/assets/sequential.png)

Answer:  a is sequentially consistent, b is not.  a can be reordered to P2:
W(x)b, P3: R(x)b, P4: R(x)b, P1: W(x)a, P3: R(x)a, P4: R(x)a

If you truly have no other channels of communication between P1, P2,
P3, and P4, this might be ok.  But how can we ensure this is true?  If
you're rendering data to a browser, I'm not sure it ever is; users can
have two tabs open, or chat with each other.

# Isolation Levels

Those three consistency levels above get a lot of attention, but I
think what's really critical when programming is this idea of
isolation.  Can we write code as if there were only one copy of the
data and we had the only program running in the system?  Usually this
requirement is too strong; [this
paper](http://www.pmg.csail.mit.edu/papers/icde00.pdf) does a great
job precisely defining different levels of isolation.

## Serializability, or Nope Still Confusing!

Most databases provide a _maximum_ isolation level of serializability.
This is often the best guarantee you can get (though as an aside,
usually when serializability is implemented using locking you also get
strict serializability).

The definition of Serializability from Wikipedia (bear with me):

> Serializability of a schedule means equivalence (in the outcome, the
> database state, data values) to a serial schedule (i.e., sequential
> with no transaction overlap in time) with the same transactions.

This says A serial schedule.  Just to be clear: This definition says
NOTHING about the order in which the transactions are issued.
Apparently the client is just shooting off transactions into the
ether, and the results of their execution will reflect SOME legal
ordering, and that's that.

So a client can issue T1, receive an "ACK", and then issue T2.  T2 is
not required to observe the results of T1. Under serializability, the
server is allowed to REORDER T2 before T1 if it feels like it, client
be damned.

A funny but true example: under serializability, the database can
execute a string of transactions sent by the client in reverse order
if it wants, as long as the results it returns to the client are legal
with respect to that ordering.

### Linearizability

This is basically the only thing that really makes sense in the world.
You perform transactions, in order, and they happen atomically and
match up with time.

# Quiz Time

Can you answer these questions correctly?  In all cases the described
operations are the only ones happening in the system.

1.  We have a database where x=0. Client A (the only client) issues
    transaction T1, which writes x=1 and returns to the client
    successfully.  The client waits until after T1 has completed, and
    now issues T2, which reads x.

    What are legal values that could be returned by T2's read of x under serializability?

    a.  0  
    b.  1  
    c.  both  
    d.  neither  

2.  We have a datastore where x=0. Client A issues a write x=1 which
    returns to the client with SUCCESS. After Client A sees the
    SUCCESS, it issues a of read x.

    What are the legal values that could be returned by T2's read of x  under sequential consistency?

    a.  0  
    b.  1  
    c.  both  
    d.  neither  

3.  What if instead of Client A issuing the read, it's Client B
    (Client B starts T2 after T1 returns by wall clock time)?

    a.  0  
    b.  1  
    c.  both  
    d.  neither  

4.  What are the legal values that could be returned by Client B's
    transaction T2 if the datastore operated with strict consistency?

    a.  0  
    b.  1  
    c.  both  
    d.  neither  

The answers:  1: c, 2: b, 3:c, 4:b

Are any of the answers surprising to you?  #1 certainly was to me
before I really started delving into the definitions.  I was aware that
a serial ordering need not reflect time or what an outside observer
might see, but I was surprised to realize that even under
serializability the order in which the client made the requests meant
nothing.
