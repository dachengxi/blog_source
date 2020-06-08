---
title: RocketMQ中的订阅关系一致
date: 2018-01-22 20:58:04
categories: 
	- MQ
tags:
	- RocketMQ
---

RocketMQ的订阅关系一致学习。

<!--more-->

订阅关系一致是指同一个消费者组下所有的消费者实例处理逻辑必须完全一致，所有消费者订阅的Topic以及Topic中的Tag必须一致。如果订阅关系不一致，消费逻辑就会混乱，导致消息丢失。



# 参考

- [https://www.alibabacloud.com/help/zh/doc-detail/43523.htm]( https://www.alibabacloud.com/help/zh/doc-detail/43523.htm)