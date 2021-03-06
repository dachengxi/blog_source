---
title: Kafka中的事务
date: 2018-05-25 21:44:30
categories: 
	- MQ
tags:
	- Kafka
---

Kafka的事务学习。

<!--more-->

消息中间件一般有三种交付语义：

- at most once，最多一次，消息可能会丢失，但是不会重复
- at least once，最少一次，消息不会丢失，但可能会重复
- exactly once，恰好一次，消息肯定并且只会被传输一次

Kafka的消息生产者提供的交付语义是at least once：

- 消息不会丢失：一旦消息被提交到日志文件，由于有多副本机制存在，消息就不会丢失
- 消息可能重复：发送消息后由于网络问题，生产者可能会重试消息发送，会导致消息重复写入

消费者交付语义和消费者处理消息提交消费位移的顺序有关：

- 消费者拉取完消息后，先处理消息再提交消费位移，可能在提交位移之前宕机，消费者重新上线后会从上一次位移提交位置拉取消息，出现重复消费，这对应了at least once
- 消费者拉取完消息后，先提交消费位移再处理消息，提交完位移后宕机，重新上线后，会从已经提交的位移处重新消费，发生消息丢失，这对应了at most once

Kafka从`0.11.0.0`开始引入幂等和事务，来实现exactly once semantics（EOS）。

# 幂等

生产者幂等性：

- 每个新生产者实例在初始化的时候会被分配一个PID（Producer Id）
- 每个PID发送消息到每个分区都有对应的序列号，序列号从0开始单挑递增，生产者每发送一条消息，就会将`<PID, 分区>`对应的序列号加1。序列号会被保存在日志中，即使分区Leader副本挂掉，新选出来的Leader也能执行消息去重。

Broker端：

- Broker会在内存中为每一对`<PID, 分区>`维护一个序列号，收到消息时，只有当他的序列号值比Broker中维护的序列号值大1，才会接收该消息。

Kafka幂等只能保证单个生产者中单分区的幂等。

# 事务

幂等性不能跨分区，事务可以弥补，事务保证对多个分区写入操作的原子性。

为了实现事务，生产者应用程序必须提供一个唯一的transactionId。

消费者事务保证的语义比较弱，Kafka不能保证已提交的事务中所有消息都能被消费：

- 采用了日志压缩策略，事务中某些消息，比如相同key的可能会被清理掉
- 事务消息可能在同一分区的多个日志段中，老的日志段被删除时，对应消息可能会丢失
- 消费者可通过seek方法访问任意offset消息，可能漏掉事务中部分消息
- 消费者消费时，可能没有分配到事务内所有分区，就不能读取事务中所有消息

日志文件中还有一种消息，叫做控制消息，一种有两种类型：COMMIT和ABORT，用来表示事务已经成功提交或被成功终止。消费者可通过这个消息来判断对应事务是被提交还是被终止了。

Kafka还引入了事务协调器（TransactionCoordinator）来负责处理事务，每个生产者都会被指派一个特定的TransactionCoordinator，所有事务逻辑包括分派PID都由事务协调器来负责，事务协调器会把事务状态持久化到内部Topic`__transaction_state`中。

流程：

- 查找事务协调器
- 获取PID
- 开启事务
- 执行
- 提交或终止事务

## 查找事务协调器

生产者会先找到事务协调器所在的Broker，Kafka收到FindCoordinatorRequet请求后，根据transactionId查找到对应的事务协调器节点，找到分区以及此分区的Leader副本所在Broker节点。

## 获取PID

找到事务协调器节点后，需要为生产者分配一个PID。事务协调器收到InitProducerIdRequest 后，会把transactionId和对应的PID以消息的形式保存到Topic`__transaction_state`中。

## 开启事务

生产者通过beginTransaction方法开启事务，生产者本地会标记已经开启了一个新事务，生产者发送第一条消息后，事务协调器才认为事务已经开启

## 执行

- 生产者给新分区发送消息前，会先向事务协调器发送AddPartitionsToTxnRequest请求，事务协调器会把`<transactionId, TopicPartition>`对应关系存储在主题`__transaction_state`中
- 生产者发送消息
- 生产者调用sendOffsetsToTransaction方法，可以再一个事务批次里处理消息的消费和发送，会先向事务协调器节点发送AddOffsetsToTxnRequest，事务协调器会将分区保存在`__transaction_state`中；之后生产者会发送TxnOffsetCommitRequest请求给事务协调器，将本次事务中包含的位移信息保存在主题`__consumer_offset`中。

## 提交或终止事务

生产者可调用commitTransaction方法或abortTransaction方法来结束当前事务：

- 生产者发送EndTxnRequest到事务协调器节点，事务协调器会将PREPARE_COMMIT或PREPARE_ABORT消息写入`__transaction_state`主题中
- 事务协调器向事务中各个分区的Leader节点发消息，通过WriteTxnMarkerRequest请求将COMMIT或ABORT写入用户所使用的普通主题和`__consumer_offsets`中，写入的是控制消息，用来标识事务的终结。
- 将COMPLETE_COMMIT或COMPLETE_ABORT写入到主题`__transaction_state`中

# 参考

- Apache kafka实战
- 深入理解Kafka：核心设计与实践原理

