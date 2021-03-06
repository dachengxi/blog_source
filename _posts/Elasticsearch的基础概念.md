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
- 节点，是一个集群中的一个服务器，可以通过配置集群名称的方式来加入一个指定的集群，有三类：数据data节点，持有数据，提供对数据的搜索功能；主节点master，负责控制其他结点工作，一个集群只有一个主节点；部落结点tribe，可以像桥梁一样连接起多个集群，允许在多个集群上执行类似在单个集群上的功能
- 索引，文档集合，数据结构是倒排索引。一个索引由一个名字来标识，必须全部小写字母，集群中可定义任意多的索引，ES内部使用Lucene写入或搜索数据
- 类型，在一个索引中，可以定义一种或多种类型，类型是索引的一个逻辑上的分类或分区，通常会为具有一组共同字段的文档定义一个类型
- 文档，是一个可被索引的基础信息单元，文档由字段构成。
- 分片，创建索引的时候可指定分片数量，分片本身是一个功能完善且独立的索引。分片允许水平分割扩展，分片上可进行分布式并行的操作提高性能和吞吐量
- 副本，分片的一份或多份拷贝，副本不和主分片在同一个节点，保证高可用。
- 映射，mapping，存储分析链所需要的信息，写入索引的时候可按照映射来存储。

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

# 字段类型

## 核心类型

| 类型       | 具体类型                                                     |
| ---------- | ------------------------------------------------------------ |
| 字符串类型 | string、text、keyword                                        |
| 数字类型   | long、integer、short、byte、double、float、half_float、scaled_float |
| 日期类型   | date                                                         |
| 布尔类型   | boolean                                                      |
| 二进制类型 | binary                                                       |
| 范围类型   | range                                                        |

- string，es5.x之后不再支持string，由text或keyword取代
- text，要被全文搜索，字段内容会被分析，被分词器分成词项，生成倒排索引；text类型字段不用于排序，很少用于聚合
- keyword，适用于索引结构化的字段，通常用于过滤、排序、聚合，keyword类型字段只能通过精确值搜索到
- 数字类型，尽可能选择范围小的数据类型，字段长度越短，索引和搜索的效率越高
- date，es内部使用长整型毫秒数存储
- boolean，true、false
- binary，接受base64编码的字符串

## 复合类型

| 类型     | 具体类型 |
| -------- | -------- |
| 数组类型 | array    |
| 对象类型 | object   |
| 嵌套类型 | nested   |

- array，es没有专用数组类型，一个数组中值必须是同一种类型
- object，JSON具有层级关系，文档内部包含对象，内部对象还包含内部对象，但是写入到es后文档就会被索引成简单的扁平key-value对
- nested，是object类型的一个特例，当对象数组独立索引和查询，Lucene没有内部对象概念，es将对象层次扁平化，转化成字段名字和值构成的简单列表

## 地理类型

| 类型     | 具体类型  |
| -------- | --------- |
| 地理坐标 | geo_point |
| 地理图形 | geo_shape |

- geo_point，存储地理位置信息的经纬度，可查找一定范围内的地理位置；通过地理位置或相对中心点距离来聚合文档；把距离因素整合到文档评分中；通过距离对文档排序
- geo_shape，可以存储一块区域，比如矩形、三角形等，GeoJSON是一种对各种地理数据结构进行编码的格式

## 特殊类型

| 类型         | 具体类型    |
| ------------ | ----------- |
| IP类型       | ip          |
| 范围类型     | completion  |
| 令牌计数类型 | token_count |
| 附件类型     | attachment  |
| 抽取类型     | precolator  |

- ip，存储IPv4或IPv6的地址
- range，range类型使用场景包括时间选择表单、年龄范围选择表单等
- token_count，用于统计字符串分词后的词项个数，本质上是一个整形字段

# 元字段

元字段是映射中描述文档本身的字段。

## 文档属性的元字段

| 具体属性 | 作用                         |
| -------- | ---------------------------- |
| _index   | 文档所属索引                 |
| _uid     | 包含`_type`和`_id`的复合字段 |
| _type    | 文档的类型                   |
| _id      | 文档id                       |

- _index，多索引查询时，支持对索引名进行term、terms查询、聚合分析、使用脚本和排序。是一个虚拟字段
- _type，可根据该元字段进行查询、聚合、脚本和排序
- _id，可用于term查询、terms查询、match查询、query_string查询、simple_query_string查询，但不能用于聚合、脚本、排序
- `_uid`，是`_type`和`_id`的组合，取值为`{type}#{id}`

## 源文档的元字段

| 具体属性 | 作用                 |
| -------- | -------------------- |
| _source  | 文档的原始JSON字符串 |
| _size    | _source字段的大小    |

## 索引的元字段

| 具体属性     | 作用                       |
| ------------ | -------------------------- |
| _all         | 包含索引全部字段的超级字段 |
| _field_names | 文档中包含非空值的所有字段 |

## 路由的元字段

| 具体属性 | 作用                               |
| -------- | ---------------------------------- |
| _parent  | 指定文档间的父子关系               |
| _routing | 将文档路由到特定分片的自定义路由值 |

## 自定义元字段

| 具体属性 | 作用             |
| -------- | ---------------- |
| _meta    | 用于自定义元数据 |

# 映射参数

- analyzer，用于指定文本段的分词器，对索引和查询都有效
- search_analyzer
- normalizer，用于解析前标准化配置，比如把所有字符转化为小写
- boost，用于设置字段的权重
- coerce，用于清除脏数据
- copy_to，用于自定义_all字段，可以把多个字段的值复制到一个超级字段
- doc_values，为了加快排序、聚合操作，在建立倒排索引的时候，额外增加一个列式存储映射，是一种空间换时间的做法
- dynamic，用于检测新发现的字段
- enabled，ES默认会索引所有字段，有些字段只需要存储，没有查询和聚合的需求，可以使用该参数来控制
- fielddata，text字段在查询时会生成一个fielddata数据结构，在字段首次被聚合、排序或者使用脚本的时候生成
- format，用于指定日期格式
- ignore_above，用于指定字段分词和索引的字符串最大长度，超过最大长度会被忽略，只用于keyword类型
- ignore_malformed，可忽略不规则数据
- include_in_all，用指定字段的值是否包含在_all字段中
- index，指定字段是否索引，不索引就不可搜索
- index_options，控制索引时存储哪些信息到倒排索引中
- fields，可让同一字段有多种不同的索引方式
- norms，用于标准化文档，以便查询时计算文档的相关性
- null_value，可让值为null的字段显式的可索引可搜索
- position_increment_gap，为支持近似或短语查询，text类型字段被解析的时候会考虑词项的位置信息
- properties，类型的映射、普通字段、object类型和nested类型字段都称为properties
- similarity，用于指定文档评分模型
- store，可设置不存储
- term_vector，词向量

# 参考

- 从Lucene到Elasticsearch全文检索实战
- 深入理解ElasticSearch

