---
title: MySQL中InnoDB的MVCC
date: 2018-04-16 22:59:10
categories: MySQL
tags: 
	- MySQL


---

MySQL的InnoDB引擎的MVCC学习。

<!--more-->

# 当前读

当前读就是加锁读，读取的是记录的最新版本，MySQL会加锁保证其他并发事务不能修改当前记录，直到当前事务释放锁。

当前读的操作主要包括：

- 显式加锁的读操作：`select ... lock in share mode;`和`select ... for update`
- 写操作insert、update、delete

update语句执行时，MySQL server会根据where条件读取第一条满足的记录，然后InnoDB会将第一条记录返回并加锁，MySQL server收到这条加锁记录后，会再发起一个update请求更新这条记录，这条记录操作完成后会继续读取下一条记录直到完成所有记录。update操作内部包含了当前读。

delete操作和update操作一样。

insert操作可能会触发Unique Key的冲突检查，也会有当前读操作。

# 快照读

快照读是不加锁读，读取记录的快照版本，而不是记录的最新版本，使用MVCC实现。

InnoDB默认的事务隔离级别Repeatable Read，读操作是快照读，如果显式加了锁的读操作不是快照读，是当前读。保证了事务执行过程中只有第一次读之前提交的修改和自己的修改可见，其他事务的修改当前事务均不可见。

# MVCC

MVCC：多版本并发控制，读不加任何锁，读写不冲突，多操作多于写操作的应用可极大增加系统并发性能。

InnoDB默认隔离级别Repeatable Read的不加锁的读操作是快照读，使用MVCC实现。

# 参考

- [https://segmentfault.com/a/1190000014133576](https://segmentfault.com/a/1190000014133576)