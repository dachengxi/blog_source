---
title: RocketMQ中的rebalance
date: 2018-01-23 15:39:27
categories: 
	- MQ
tags:
	- RocketMQ
---

RocketMQ的rebalance学习。

<!--more-->

rebalance场景：

- 消费者发送心跳到Broker，Broker端发现有新的消费者进来或者新增了topic订阅信息或者删除了topic订阅信息，Broker会通知所有消费者NOTIFY_CONSUMER_IDS_CHANGED，消费者收到请求后会立刻进行rebalance：MQClientInstance#rebalanceImmediately



# 参考

