---
layout: post
title:  "System Design"
date:   2025-01-02 00:02:29 -0400
categories: jekyll update
---

Click [here][video-link] for source video.


## Design a system that does counting at a large scale

The requirement can be to count the YouTube video views, or count the number of likes on Facebook. The problem can be more vague, and ask to calculate some application performance analytics, we don't know.

<br/>

### Why is requirement clarification important?

The interviewer wants to know how the candidate deals with ambiguity. This is important, because this trait is important for a software engineer's daily job. The engineer should know the functional pieces they're going to design for the rest of the interview.

For the interviewee, there can be many approaches to solve the problem, but each approach has its pros and cons. We should pick those approaches which address the requirements. The engineer can be an expert in database design, or in cloud computing, etc., and they can provide different ways to solve this counting at a large scale problem.

<br/>

### Requirement clarifications

There are 4 categories that we should focus on:
1. Users/Customers: We want to understand who and how the system will be used.
2. Scale (read and write): We want to understand how our system will handle growing amounts of data.
3. Performance: We want to know how fast our system must be.
4. Cost: We need to understand budget constraints.

#### Users/Customers:
1. Who will use the system? Is it for all the YT viewers, or is it only visible to the author, or is it to be used for the training of a ML model?
2. How will the system be used? Is the data to be streamed real time, or is it going to be stored for future use? This helps us understand how the data will be used.

#### Scale (read and write):
1. How many read queries per second does the system need to process?
2. How much data is queried per second?
3. How many video views are processed by the system per second?
4. Can there be spikes in traffic, and if so, how big can they be?

We need to ask these questions, or we can assume some reasonable values.

#### Performance:
The questions asked in the category should help us quickly evaluate different design options.
1. What is the expected write to read data delay? (i.e., do we need to count views in real time, or can we do it at a later stage using batch processing.)
2. What is the expected p99 latency for read queries?

#### Cost:
1. Should the design minimize the cost of development? (use of open source software.)
2. Should the design minimize the cost of maintainence? (considering use of public cloud servers.)



[video-link]: https://www.youtube.com/watch?v=bUHFg8CZFws