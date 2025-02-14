---
layout: post
title:  "Distributed Systems"
date:   2024-01-01 00:02:29 -0400
categories: jekyll update
---

Revisiting distributed systems concepts.

<br/>

## Part 1 - Introduction, time, and clocks
<em>1/18/2025</em>

There are various definitions for distributed systems. A distributed system runs on several `nodes`, connected by a network. They are characterized by `partial failures`.

Partial failures occur when some parts of a system fail while others continue to work. These situations are particularly challenging because software often struggles to identify which parts are broken. Ideally, distributed systems should distribute the workload fairly or operate as though they are a single cohesive system, maintaining transparency. However, these goals are often overly optimistic.

A common example of partial failure is when a network of machines experiences a failure due to a network issue. Failures can also occur because of software misbehavior. There are additional perspectives on partial failure as well. For instance, on a single machine, if a part of the operating system fails, it can lead to a kernel panic, requiring the machine to be restarted to recover. In a system with multiple machines, if one machine fails, it becomes critical to ensure the rest of the system continues to function. Restarting all the other machines just because one has failed would not be practical. This philosophy of maintaining system functionality despite partial failures is at the heart of cloud computing.

Cloud computing is specifically designed to handle partial failures by working around them. On the other hand, high-performance computing (HPC) takes a different approach, treating partial failures as complete failures. In HPC, if something goes wrong, the computation is restarted entirely. To minimize the impact, checkpointing is used so the process can resume from the point of failure. Despite these differences in approach, HPC is still a type of distributed system.

In addition to machines themselves failing, the connection between them can fail as well. Let's say there are 2 machines. Machine M1 sends a request to machine M2 requesting a value for X. What are some of the things that could go wrong?
```
1. Request sent from M1 gets lost.
2. Request from M1 is very slow. M2 can't wait forever.
3. M2 has crashed.
4. M1 sends a message, but M2 is taking a lot of time to compute.
5. M2 sends a response, but it is very slow.
6. M2 sends a response, but the message gets lost.
7. M2 could be a bad actor - it could lie or refuse to answer.
8. M2's response gets corrupted.
```

Does M1 have any way of knowing what happened in the above situations? No. From M1's perspective, all these situations are indistinguishable. If a machine sends a request to another node, and it doesn't receive a response, it is impossible to know why (without the global knowledge of the system, which we don't know).
The last 2 causes are known as Byzantine faults.
There can be another case where the value sent by M2 is stale by the time it reaches M1. This is a very important topic of discussion.

How do real systems deal with such cases? Machines need to have some kind of timeout. After a certain amount of time, if we don't get a response, we assume that some failure has taken place. But is it correct to assume failure? What if M1 caused a side effect on M2? M1 asked to increment a value to M2, which M2 does, and sends an OK response. If M1 gets the response, it can assume that the value was incremented. However, if M1 doesn't receive the OK response, it would be wrong to assume that the increment didn't occur. This sort of uncertainty is a fundamental aspect of distributed systems.

So do timeouts really work?

One of the biggest causes of uncertainty in distributed systems is network latency. If we know the maximum time it takes for a message to travel from one machine (M1) to another (M2), represented as d, and we also know the maximum time M2 might take to process a request, represented as r, we can calculate a timeout using the formula `(2d + r)`. This calculation helps eliminate some of the uncertainty caused by delays or slowness in the system.

However, even with this formula, many uncertainties remain. There is no guarantee that the maximum delay value we use is accurate, as network conditions can change unpredictably. This means that, in addition to managing partial failures, distributed systems must also handle the challenges of unbounded latency. In essence, a distributed system can be described as a combination of partial failures and unbounded latency, both of which must be addressed for the system to function effectively.

Why would we want a distributed system, given it has so many issues? Data can be too big to fit on a single machine. There can also be a need to make tasks faster, which can be done with more computers.

<br/>

### Time and clocks

What do we use clocks for in computing? We use clocks to say when something would happen so that we can rendezvous. Other main reasons include:
1. It is used to mark `points` in time. For example, "The class starts at 9:20 AM PDT."
2. Timeouts are for discussing `durations` or `intervals` of time. For example, "This class is 65 mins long."

Computers have 2 types of clocks - time-of-day (TOD) clocks and monotonic clocks.

<br/>

#### Time-of-day clocks:
Time-of-day clocks are synchronized using NTP (Network Time Protocol). However, they are not suitable for measuring durations or intervals of time. One key reason is that these clocks may not be perfectly synchronized across systems. Additionally, time-of-day clocks can jump forward or backward, such as during daylight savings time, which makes them unreliable for precise measurements.

When it comes to timestamping specific events, time-of-day clocks are somewhat useful, but they are far from ideal. The accuracy of clock synchronization has its limits, and we often need a much finer resolution to prevent certain types of bugs. This lack of precision makes them less reliable in scenarios requiring exact timing.

<br/>

#### Monotonic clocks:
These clocks only go forward. It is going to be some sort of a counter, which stores milliseconds from the time the machine was started. This would be different for different machines. Its absolute value is meaningless. So, it is bad for the purpose of timestamping but good for calculating durations and intervals. Unix timestamps are monotonic clocks. Below is a Python code example:
```
import time
time.monotonic()
```

So, which of these clocks are useful in distributed systems? We can say monotonic clocks, since they can be used to implement timeouts. (Read more about the Cloudflare leap second bug - implement timeouts using time-of-day clocks.)

The above 2 clocks we discussed are `physical` clocks. In distributed systems, we need a different notion of clocks, which is called a logical clock.

<br/>

#### Logical clocks
They only measure the `ordering of events`.

A fundamental question when talking about ordering of events is, which event happened before another? This will be particularly important to know (example - the case where a 3rd machine was updating the value of X). It is extremely crucial to know in distributed systems which event happened before another. They can be anything, like database writes, etc.

Representing this notion, suppose A happened before B. The expression would look like:
```
A -> B
```

What does this tell us about causality (what could have caused what)? If A happened before B, we know for sure A `could` have caused B. But B `could not` have caused A. This is important and could help us in debugging. It will at least allow us to rule out something.

Things that `happens-before` reasoning is good for:
1. Debugging
2. It is useful in designing the system in the first place.

<br/>

<br/>

## Part 2 - Lamport diagrams, causality, and happens-before relation
<em>1/23/2025</em>

Earlier, we expressed the notion of happens-before relation as `A->B`, which means A happened before B. But looking at the machine diagrams, it doens't really tell the full picture. We will be using Lamport diagrams (also called space time diagrams).

A process is just a line with a discrete time beginning. Events will be dots on this line. Let's have a look at an example:

![image tooltip here]({{ "/assets/distributed_systems/001_timing_diagram_main.jpeg" | relative_url }})

<br/>

Could X have caused Z? Yes. Do we know fo sure? No. Could Z have caused X? No.

Machines don't share memory, so the only way they would know each other's state would be by exchanging messages.

From the above diagram, events P, T, and U are message sent events. Events S, Q, and R are message received events. X, Y, Z, and V are called internal events.

Let's try to have a picture of events happening before other events. Below is the definition of the happens-before relation:
Given 2 events A and B, we say `A->B` (A happened before B) if any of the following are true:
1. A and B occu on the same process line, with B after A.
2. A is a message sent event, and B is the corresponding recieve.
3. If `A->C`, and `C->B`, then `A->B` (transitive closure of the relation.)

The above rules are not an algorithm.

<br/>

#### Classic Lamport diagram

![image tooltip here]({{ "/assets/distributed_systems/002_alice_timing.jpeg" | relative_url }})

In the above image, Alice's message takes time to reach Carol. When Bob sends its message, it appears before Alice's message. Carol is confused, and this is called a causal anomaly.

Alice's message took a while to reach Carol. Previously, we discussed unbounded latency. If this is not handled, it would lead to bugs.

<br/>

#### Concept of network models

It would be helpful to have some information about some fixed amount of time it takes for messages to travel between systems. A network which consists of such type of systems is called a synchronous network. A synchronous network is the one where there exists an `n`, such that no message takes longer than `n` units of time to be delivered.

More time will be spent learning about another network model, called the asynchronous network. It is a network where there exist no such `n`.

<em>Notes: look up partially synchronous network, Lynch-distributed algorithms; do synchronous network models work for large value of n; for a model, we make a set of assumptions; impossibilty test, distributed algorithms test.</em>

<br/>

#### States and Events

![image tooltip here]({{ "/assets/distributed_systems/003_process_rep.jpeg" | relative_url }})

What is a state a machine can be in? (comparing with computer architecture, we determine state of a machine by looking at the contents of memory in register, and so on.)

A point is an event, but it also represents a state. In our above image, there will be a point before when X was set to 5. X = 5 represents an event that took place which set X as 5. A sequence of events is boiled down to a state.

Is it possible to work out what sequence of events lead to a state?

<br/>

#### Recap of the happens-before relation

Given A and B are events:
* if A and B occur on the same process line with B before A, then `A->B`.
* if A is a send, and B is the corresponding receive, then `A->B`.
* if `A->C`, and `C->B`, then `A->B`.

<br/>

![image tooltip here]({{ "/assets/distributed_systems/004_happens_before_diagram.jpeg" | relative_url }})

Let's breakdown what's happening in the above image:
1. From the 2 process line diagram, we can describe the below happens-before relations: `X->Y, Y->Z, X->Z, Z->Q, X->Q`. What are the smallest relations among the above relations?
2. For the 3 process line diagram, we can make the following set: `{(A, B), (B, C), (A, C), (D, C)}`. We don't know about `(A, D)` for sure. Note that our definition of happens-before does not give this information.

The point is, we can throw as much garbage of happens-before relations, which checks out with the diagram, but we don't want those relations. We need to find the smallest set such that all the points are included.

<br/>

We refer to our first image in this section. We can make the following happens-before relations:
`{(P, S), (X, Z), (P, Z)}`.

`(Q, R)` cannot be expressed using the happens-before relation. So, `Q->R` is false, and `R->Q` is also false. If this happens for 2 events, we say that `Q` and `R` are concurrent, which can be expressed as `Q||R`. Another way of saying they are concurrent is that they are independent.

<br/>

#### Partial orders

This happens-before relation is a special kind of partial order relation:
It is a set S, that comes together with something ...
A binary relation, usually, but not always, written as `<=`, that let's you compare things from the set S, ad it needs to have the followjng properties:
1. Reflexivity - it means that, for all a belonging to S, `a<=a`.
2. Antisymmetry - for all a and b belonging to S, if `a<=b` and `b<=a`, then `a=b`.
3. Transitivity - for all a, b, c in the set S, if `a<=b`, and `b<=c`, then `a<=c`.

How do we transition these partial order properties to a distributed system?

The happens-before relation doesn't reflect on the partial order rules. Happens-before is irreflexive. Certainly, transitivity is going to be true. What about antisymmetry? In some sense it checks out. It is vacuously true.

<br/>

<br/>

## Part 3 - Recap of partial orders, total order, and happens-before
<em>1/27/2025</em>

What are distributed systems? We went through some definitions in Part 1.

We also saw the properties associated with partial orders. For our use of happens-before relation to observe distributed systems, the property of reflexivity does not hold. We would call `A->B` as an irreflexive partial order. The antisymmetry and transitive properties hold true.

<em>Refer notes for set inclusion, total order.</em>

In Part 1, we discussed the notion of logical clocks. One of the representation are Lamport clocks (LC). For events `A` and `B`, if `A->B`, then `LC(A) < LC(B)`.
Lamport clocks are consistent with causality.

### Assigning LC to events:
1. Every process has a counter, intially set to 0.
2. On every event, a process increments its counter.
3. When sending a message, a process includes its current counter along with the message.
4. When receiving a message, set the counter to the max(local counter, message counter) plus 1.

<br/>

![image tooltip here]({{ "/assets/distributed_systems/005_lamport_clock_1.jpeg" | relative_url }})

Let's refer the above image. For all the processes, the counter is set to 0. When events occur on the process line, counter is incremented. On process line B, it  receives a message from process line A. The local counter is 0, and message counter is 1. Applying the 4th rule, the LC for that event will be set to 2. As LC are updated, the LC for the process line is updated as well. For the message received on C, the LC will be updated to 4.

So if we observe, when `A->B`, then `LC(A) < LC(B)`.

<br/>

We know if `A->B`, then `LC(A) < LC(B)`. If we check the reverse, that is, if `LC(A) < LC(B)`, do we know if `A->B`? We don't. Let's look at the below example to understand this property.

![image tooltip here]({{ "/assets/distributed_systems/006_lamport_clock_2.jpeg" | relative_url }})

Every process starts with 0. We have drawn the LC for all the events by referring the 4 rules. We also assume that all the events occur (there is not case of event failure, we'll talk about this in later parts.)
Now, if we look at event A and B, we see that `LC(A) = 1`, and `LC(B) = 5`. Do we know for sure if `A->B`? The answer is a resounding NO. This is because to represent events with a happens-before relation, we need 1 of 3 cases to hold. They are on different process lines. They are not sending or receiving messages between themselves. There is no transitive relation between them. All the partial order relations fail for events A and B. For this reason, we cannot say if A happened before B.

There is no way to draw a line that goes from A to B.
**Causality is graph reachabilty in space time.** What it means is, if we cannot reach B from A, then we cannot say that `A->B`, even if `LC(A) < LC(B)`.

<br/>

**Lamport clocks (LCs) are consistent with causality.** But it's not the case that if `LC(A) < LC(B)`, then `A->B`.
LCs do not **characterize** causality.

<em>Previous image taken from Schwarz and Mattern (1994) - Detecting causal relationships in distributed systems.</em>

<br/>

What are LC good for then? We can construct a mathematical argument from it. We know if `A->B`, then `LC(A) < LC(B)`. What is the contrapositive for this expression?

If `¬(LC(A) < LC(B))`, then `¬(A -> B)`
These can be used to rule out things as not having caused other things. This turns out to be very valuable, and can be useful for debugging.

<br/>

Even if A happened before B in terms of a physical clock, we still can't establish that using the happens-before relation. Because there is no concept of global clock, we need to make relations between systems using the happens-before relation to observe what's happening.

<br/>

<br/>

### Part 4 - Vector clocks, protocol rules and anomalies

In the previous part, we discussed the relation between Lamport clocks (LC) and the happens-before relation:
`if A->B, then LC(A) < LC(B)`

But we also saw that the reverse was not true, i.e.:
`if LC(A) < LC(B), we cannot say A->B.`

<br/>

#### Vector clocks

If `A->B`, then `VC(A) < VC(B)`. Also, if `VC(A) < VC(B)`, then `A-B`.

Vector clocks are consistent with and characterize causality.

A vector clock is just a sequence of integers:
1. Every process maintains a vector of integers initialized to 0, one for every each process.
2. On every event, a process increments its own position in its vector clock. <em>(Vector clocks only work if we have the knowledge of how many processes are running.)</em>
3. When sending a message, each process includes its current clock (after incrementing its own position, since send is an event.)
4. When receiving a message, each process updates its vector clock to the max of the received vector clock and its own (after incrementing its own clock, since receive is an event.)

How is the max calculated? - Pointwise maximum. Below is an example:
For vectors `[1, 12, 4]` and `[7, 0, 2]`, the pointwise max would be `[7, 12, 4]`.

How do we know if one VC is less than the other?

From the above happens-before relation, we expressed the relation for VC as follows: if `A->B`, then `VC(A) < VC(B)`. This goes both ways. How do we determine the result for `VC(A) < VC(B)`?

```
VC(A) < VC(B) when,
VC(A)i <= VC(B)i, where i subscript stands for all i,
and VC(A) != VC(B).
(There has to be at least one number that is strictly smaller.)
```

For example, if `VC(A) = [2, 2, 0]` and `VC(B) = [3, 2, 0]`, we can say that `VC(A) < VC(B)`.

If `VC(A) = [2, 2, 0]` and `VC(B) = [1, 2, 3]`, VC(A) is not less than VC(B). Also, VC(B) is not less than VC(A). In this case, the events A and B are considerd concurrent or independent.

<br/>

#### Below is a Lamport diagram with vector clocks:

![image tooltip here]({{ "/assets/distributed_systems/007_vector_diagram.jpeg" | relative_url }})

<em>All of the processes have to agree on the order in the vector clock.</em>

We'll make observations from the above image. What is in the causal history of event A? How does the vector clock of A compare with the VC of B? `A = [2, 4, 1]` and `B = [0, 3, 2]`. We can see that the VC(A) is neither bigger not smaller than the VC(B). Thus we can say that events A and B are independent, or causally unrelated, or concurrent.

Instead of thinking of graph reachabiltiy, we can instead compare these vectors. Computers are good at comparing numbers. Using this data, we can exactly capture the happens-before relation. There would be a lot of other events that are going to be concurrent with A.

<br/>

#### Protocol

It is a set of riles that computers use to communicate with each other.

![image tooltip here]({{ "/assets/distributed_systems/008_protocol_1.jpeg" | relative_url }})

From the above image, let's say that we define a protocol - when the first process asks a message, the other process must reply with a message. We have Alice on one process, and Bob on the other process communicating with each other.

The first 2 diagrams show no protocol violation. The 3rd diagram is a protocol violation, since Alice replied with a message before Bob asked it. Is the protocol being violated for diagam 4? We don't know, because it still might be running. As seen in the below image, Alice may be slow to respond.

![image tooltip here]({{ "/assets/distributed_systems/009_protocol_2.jpeg" | relative_url }})

Distance doesn't mean anything in the Lamport diagram.

There are infinitely correct runs for a protocl using Lamport diagrasms. We instead can draw up violations of the protocol.

<br/>

#### FIFO delivery

If a process sends message M2 after message M1, any process delivering both, delivers M1 first.

We'll discuss FIFO anomaly in the next part.

<br/>

<br/>

### Part 5 - Recap of delivery vs receiving, recap of FIFO delivery.
<em>2/4/2025</em>

#### Receiving vs delivery of messages:
Sending a message is an action performed by a system. Receiving a message is something that happens to the system. However, delivering the message to the system is an action the system itself can take.

Assume there are two process lines, A and B. A sends a message to B. At B, before the message can be delivered to the process, it is queued up. A protocol may wait to deliver the message at a later time. This can be important because some messages might be sent out of order and need to be queued on the receiver's side before being processed.

<br/>

#### FIFO delivery:
If a process sends message M2 after message M1, any process that delivers both must deliver M1 first. If M2 arrives before M1, it results in a FIFO violation. Essentially, a FIFO violation occurs when the order of messages in a queue or message pipeline is disrupted.

In distributed systems, implementing FIFO delivery manually is rarely necessary. This is because TCP enforces FIFO delivery by ensuring that messages are received in the order they were sent.

<br/>

#### Causal delivery:
If M1's send happens before M2's send, then M1's delivery happens before M2's delivery.
Referring to the example of FIFO violation, a system with FIFO delivery violation is also a causal delivery violation.
Let's see another example of Alice, Bob and Carol.

![image tooltip here]({{ "/assets/distributed_systems/010_causal_delivery_ex.jpeg" | relative_url }})

So, the above diagram is not a FIFO violation, since the sending of messages didn't happen from the same process. Different processes sent different messages, so from Carol's perspective, both messages would be M1 from Alice and Bob. The sending itself is happening from different process lines.
So vacously, this is not a FIFO delivery violation, but a causal delivery violation.

<br/>

#### Total-ordered delivery:
If a process delivers message M1, and then M2, then all processes delivering both M1 and M2 deliver M1 first.

Below is the anomaly violating this property:

![image tooltip here]({{ "/assets/distributed_systems/011_to_anomaly.jpeg" | relative_url }})

Client 1 (C1) sends messages M1 to Replicas R1 and R2. Client 2 (C2) sends messages M2 to R1 and R2 as well. However, due to the order in which the messages arrived, the replicas will have different value of X. The replicas disagree in this case. This image shows a total-order anomaly.

There is no FIFO delivery violation in the above image.

<br/>

<em>Does a system guarenting causal delivery, also give FIFO delivery? - YES.</em>

<em>A system has FIFO delivery violation. Process A sends messages M1 followed by M2, but process B receives them in the order M2, and then M1. How is this not a totally-ordered delivery violation?</em>

<br/>

#### Heirarchy of delivery guarentees:

![image tooltip here]({{ "/assets/distributed_systems/012_hierarchy_delivery.jpeg" | relative_url }})

Let's interpret the image above. **Guaranteeing causal delivery also ensures FIFO delivery.** However, a system that provides FIFO delivery does **not** necessarily guarantee causal delivery.  

Totally-ordered delivery operates differently. It branches out, meaning that a system with totally-ordered delivery does **not** guarantee FIFO or causal delivery. The same applies in reverse—a system that ensures causal or FIFO delivery does **not** necessarily provide totally-ordered delivery.

<br/>

#### Implementing FIFO delivery:

##### Approach 1:
A typical approach would be by using sequence numbers (SN). Messages get tagged with a seqence number from the sender, along with a sender ID. The sender increments its sequence number after sending. If a received message's SN is the (SN + 1) of the previously delivered message from that sender, deliver it. Otherwise, it is queued up for later.
What are some situation this approach would not work? A case would be when the message gets lost, or is very slow to reach the receiver.

![image tooltip here]({{ "/assets/distributed_systems/013_FIFO_queued.jpeg" | relative_url }})

We are assuming that there is reliable delivery.

Suppose there was no reliable delivery, how can FIFO delivery be implemented?

We can process messages that are queued if they have a higher sequence number than the previously received message. But what happens if a message we assumed was lost arrives at the receiver after a long delay?

In such cases, we can use buffers. Messages that arrive out of order are temporarily stored in the buffer (for example, messages 4 and 5), while the system waits for message 3. Once message 3 arrives and is delivered, the remaining messages can be processed in the correct order. However, determining the appropriate buffer size is a crucial design decision for the system.

<em>If all the messages received at B are dropped by B, is it violating FIFO delivery? No, because no messages are getting delivered, so it is vacously following FIFO delivery.</em>

<br/>

##### Approach 2:
We can implement FIFO delivery using acknowledgments. Assume there are two processes, A and B. A sends a message to B, and once B receives it, it sends an acknowledgment back to A. Only after A receives the acknowledgment will it send the next message. However, this approach is very slow.

<br/>

#### Implementing causal delivery:
The diagram where Alice, Bob, and Carol were involved, was a case of causal anomaly. Carol reciveies a message first that causally happened at a later time.

To address this, we can use vector clocks.
**IMPORTANT ASSUMPTION: We only have to track the message SENDS.**

![image tooltip here]({{ "/assets/distributed_systems/014_causal_delivery_vc.jpeg" | relative_url }})

Let's review the above diagram.

Alice broadcasts a message, followed by Bob. Each process is assigned a vector clock, and we assume that we will only track message sends. The vector clock for a process does not increment for any internal or receive events.

When Alice broadcasts the message, her vector clock updates to **[1, 0, 0]**. At **Marker 1**, Bob receives Alice's message, but since it is only a receive event, no change occurs in his vector clock. When Bob then broadcasts his own message, his vector clock updates to **[1, 1, 0]**.

Now, at **Marker 3**, Carol receives Bob's message. She checks the vector clock that came with the message and compares it with her own. Since the message arrived from Bob, she looks at Bob’s index and sees that it has a value of **1**, which is exactly **one more** than her own clock. So far, everything is fine. However, when she checks Alice's index, she notices that it also has a value of **1**, which is one more than her own maintained copy of the clock. But Carol realizes she never received a message from Alice. This means there is a missing message that needs to be received before Bob’s message can be delivered. As a result, Carol **queues Bob’s message and does not deliver it**.

At **Marker 2**, the system checks if it is okay for Alice's message to be delivered. In this case, it is valid.

Now, at **Marker 4**, Carol can finally deliver Bob’s message. She compares the vector clock and sees that Alice's message has a value **1 more** than her own, meaning it must have been the first message sent by Alice. 

Once this is done, she checks the buffer for any queued messages. Since Alice's message has now been delivered, Carol is able to deliver Bob’s message as well. This approach ensures **causal delivery**.

<em>This algorithm only works for causal broadcast.</em>

<br/>

<br/>