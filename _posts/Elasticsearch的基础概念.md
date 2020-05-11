---
title: Elasticsearch的基础概念
date: 2018-06-16 23:34:40
categories: 
	- Elasticsearch
tags:
	- Elasticsearch
---

Elasticsearch基础概念学习。

<!--more-->

# 核心概念

- 集群，一个或多个ES服务器节点组成集群，一个集群由一个唯一的名字标识，成为cluster name，默认名elasticsearch。具有相同集群名称的节点才会组成一个集群
- 节点，是一个集群中的一个服务器，可以通过配置集群名称的方式来加入一个指定的集群
- 索引，拥有几分相似特征的文档集合，数据结构是倒排索引。一个索引由一个名字来标识，必须全部小写字母，集群中可定义任意多的索引
- 类型，在一个索引中，可以定义一种或多种类型，类型是索引的一个逻辑上的分类或分区，通常会为具有一组共同字段的文档定义一个类型
- 文档，是一个可被索引的基础信息单元
- 分片，创建索引的时候可指定分片数量，分片本身是一个功能完善且独立的索引。分片允许水平分割扩展，分片上可进行分布式并行的操作提高性能和吞吐量
- 副本，分片的一份或多份拷贝，副本不和主分片在同一个节点，保证高可用。

# 对比RDMS

| RDMS                | Elasticsearch     |
| ------------------- | ----------------- |
| 数据库database      | 索引index         |
| 表table             | 类型type          |
| 行row               | 文档document      |
| 列column            | 字段field         |
| 表结构schema        | 映射Mapping       |
| 索引                | 全文索引          |
| sql                 | 查询DSL           |
| select * from table | GET http://...    |
| update table set    | PUT http://...    |
| delete              | DELETE http://... |



# 参考

- 从Lucene到Elasticsearch全文检索实战
- 深入理解ElasticSearch

