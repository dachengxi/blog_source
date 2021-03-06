---
title: Redis数据库实现以及各种操作实现
date: 2019-03-25 22:10:07
categories: 
	- 缓存
tags:
	- redis
---

Redis数据库实现，保存键值对的方法，针对数据库添加、删除、更新、查询等方法的实现，设置生存时间和过期时间操作。

<!--more-->

# 数据库实现

Redis服务器的所有数据库存在redisServer的db数组中：

```c
redisServer {
  // db数组，保存服务器中所有数据库
  redisDb *db;
  
  // 服务器的数据库数量
  int dbnum;
}
```

服务器初始化时，程序会根据dbnum属性来决定创建多少个数据库，默认16个。

服务器数据库示例：

![server-1](./Redis数据库实现以及各种操作实现/server-1.png)

# 数据库键空间

Redis数据库由redisDb结构表示，所有键值对保存在redisDb的dict字典中，这个字典称为键空间key space：

```c
redisDb {
  // 键空间，保存数据库中所有键值对
  dict *dict;
  
  // 过期字典，保存着键的过期时间
  dict *expires
}
```

- 键空间的键，就是数据库的键，是字符串对象
- 键空间的值，就是数据库的值，可以是：字符串对象、列表对象、哈希表对象、集合对象、有序集合对象

键空间示例：

![keyspace-1](./Redis数据库实现以及各种操作实现/keyspace-1.png)

- 添加新键：将新键值添加到键空间字典里面。
- 删除键：在键空间里面删除键所对应的键值对对象。
- 更新键：对键空间里键对应的值对象进行更新。
- 对键取值：在键空间字典里取出键对应的值对象。

## 读写键空间时的附加操作

- 对键的读写操作，会根据键是否存在来更新服务器键空间命中hit次数和不命中miss次数。
- 读取键后，服务器会更新键的LRU时间。
- 读取一个键时发现键过期，服务器会先删除这个键。
- 客户端如果使用WATCH监听了键，服务器对键的修改，会把这个键标记为dirty，让事务程序注意到这个键被修改。
- 服务器每修改一次键，会对dirty计数器加1，计数器会触发服务器的持久化以及复制操作。
- 如果服务器开启了数据库通知功能，对键修改后，服务器会按照配置发送通知。

# 设置键的生存时间或过期时间

- EXPIRE，设置键的生存时间TTL，单位：秒
- PEXPIRE，设置键的生存时间TTL，单位：毫秒
- EXPIREAT，设置键的过期时间，单位：秒
- PEXPIREAT，设置键的过期时间，单位：毫秒
- EXPIRE、PEXPIRE、EXPIREAT三个命令都是使用PEXPIZREAT命令来实现的。

## 保存过期时间

redisDb使用expires字典来保存数据库中所有键的过期时间，叫做过期字典：

```c
redisDb {
  // 键空间，保存数据库中所有键值对
  dict *dict;
  
  // 过期字典，保存着键的过期时间
  dict *expires
}
```

- 过期字典的键是个指针，指向某个键对象。
- 过期字典的值是一个long long类型整数，保存了键指向的键对象过期时间，毫秒经度的unix时间戳。

# 参考

- 《redis设计与实现》（第二版）