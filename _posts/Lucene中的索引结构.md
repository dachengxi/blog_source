---
title: Lucene中的索引结构
date: 2018-06-18 14:08:31
categories: 
	- Lucene
tags:
	- Lucene
---

Lucene中的索引结构学习。

<!--more-->

Lucene支持两种索引结构：多文件索引结构和复合索引结构。多文件索引结构使用多个文件来表示索引，复合索引结构使用一个特殊的文件来表示。

默认是用的是复合索引结构，可通过设置`useCompoundFile=false`来启用多文件索引结构。



# 多文件索引结构

通过设置`useCompoundFile=false`来启用多文件索引结构

# 参考

- Lucene实战

