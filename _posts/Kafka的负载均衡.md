---
title: Kafka中的负载均衡
date: 2018-05-20 10:39:27
categories: 
	- MQ
tags:
	- Kafka
---

Kafka的负载均衡学习。

<!--more-->

# 生产者发消息的负载均衡

生产者发消息时，会根据Partition的策略来决定将消息发往哪个分区：

- 如果消息中指定了partition字段，就往指定的这个分区中发消息
- 如果没有指定partition，需要依赖分区器，根据key来计算partition的值。可以使用Kafka默认分区器或者我们自定义分区器

## Kafka默认分区器

默认分区器是DefaultPartitioner：

- 如果key不为null，默认对key进行哈希计算，来得到分区号。可以保证相同的key被发送到相同分区上
- 如果key为null，消息将以轮询的方式发送到主题的各个可用分区。

# 消费者的负载均衡

消费者客户端可以使用`partition.assignment.strategy`来设置消费者与订阅主题之间的分区分配策略，有三种分区分配策略：RangeAssignor、RoundRobinAssignor、StickyAssignor，默认是RangeAssignor

- RangeAssignor，按照消费者总数和分区总数进行整除运算来获得一个跨度，然后将分区按照跨度进行平均分配，保证分区尽可能均匀的分配给所有消费者。会将消费组内所有订阅了这个主题的消费者按照名称字典序排序，然后为每个消费者划分固定的分区范围，如果不够平均分配，靠前的消费者会被多分配一个分区
- RoundRobinAssignor，将消费组内所有消费者以及消费者订阅的主题分区按照字典序排序，通过轮询的方式将各个分区一次分配给每个消费者。
- StickyAssignor，这总分配策略尽可能均匀分配、重分配时分配的分区尽可能与上次分配的保持相同。

# 参考

- Apache kafka实战
- 深入理解Kafka：核心设计与实践原理