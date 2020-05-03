---
layout:     post
title:      "Flink consumer and Kafka partition"
subtitle:   "Setting the parallelism and partition right"
date:       2020-05-03 14:24:00
author:     "ChenRiang"
catalog: true
tags:
    - Flink 
    - Kafka
---

# Background

Something interesting was found when running Flink job that consumes Kafka stream data as a source. **No matter how many parallelism there is always 1 task manager consuming data from Kafka**.



{% include image.html src="post-kafka-flink-consumer-issue.png" data="group" title="Flink UI Console" %}

<br>

# Findings
Flink creates consumer depends on the number of parallelism user set. When Flink consumers that created is more than Kafka partition, some of the Flink consumers will just idle!

So, the problem is in Kafka. The topic partition created by default is 1. By adding Kafka topic partitions that match Flink parallelism will solve this issue. 

<br>
There is 3 possible scenario cause by number of Kafka partition and number of Flink parallelism :

1. Kafka partitions == Flink parallelism
This case is ideal, since each consumer takes care of one partition. If your messages are balanced between partitions, the work will be evenly spread across Flink operators

2. Kafka partitions < Flink parallelism
Some Flink instances won't receive any messages.

3. Kafka partitions > Flink parallelism
Some instances will handle multiple partitions.


<br>
Learning:

> **Always make sure Kafka topic partition >= Flink parallelism.** 

<br>

# Reference 
[Kafka + Flink: A Practical, How-To Guide](https://www.ververica.com/blog/kafka-flink-a-practical-how-to)

[Kafka partitions and Flink parallelism](https://riptutorial.com/apache-flink/example/27996/kafka-partitions-and-flink-parallelism)

