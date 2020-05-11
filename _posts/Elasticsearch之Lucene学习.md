---
title: Elasticsearch之Lucene学习
date: 2018-06-16 16:01:30
categories: 
	- Elasticsearch
tags:
	- Elasticsearch
---

信息检索基础概念学习，包括分词算法、倒排索引等等。

Lucene基础学习。

<!--more-->

文档是Lucene索引基本单位，比文档更小的单位是字段，字段由三个部分组成：

- name
- type
- value

# Lucene字段类型

- TextField，字段内容会被索引并词条化，但不保存词向量
- StringField，只会对该字段内容索引，不词条化，也不保存词向量，字符串值会被索引为一个单独的词项
- IntPoint，适合索引值为int类型的字段
- LongPoint，long类型
- FloatPoint，float类型
- DoublePoint，double类型
- SortedDocValuesField，存储多值域的DocValues字段，适合索引字段值为文本并且需要按值进行分组、聚合等操作字段
- NumbericDocValuesField，存储单个数值类型的DocValues字段
- SortedNumbericDocValuesField，存储数值类型的有序数组列表的DocValues字段
- StoredField，适合索引只需要保存字段值不进行其他操作的字段

# 参考

- 从Lucene到Elasticsearch全文检索实战
- 深入理解ElasticSearch

