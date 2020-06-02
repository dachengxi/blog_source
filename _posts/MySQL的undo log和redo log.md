---
title: MySQL的undo log和redo log
date: 2018-04-18 20:32:05
categories: MySQL
tags: 
	- MySQL


---

MySQL的undo log和redo log学习。

<!--more-->

# undo log

回滚日志保存了事务发生之前的数据的一个版本，可以用于回滚。undo log还可以提供一致性读（MVCC）。

在事务开始之前，当前版本行数据生成undo log，undo log也会产生redo log来保证undo log的可靠性；事务提交之后undo log不会立马删除，会放入待清理的链表，由purge线程判断是否有其他事务在使用undo段中表的上一个事务之前的版本信息，决定是否可以清理undo log。

undo log是逻辑格式的日志，执行undo仅仅是将数据从逻辑上恢复至事务之前的状态，而不是从物理页面上操作。

undo log是为了实现事务的原子性，undo log还可以用来实现MVCC。操作任何数据前线将数据备份到undo log，然后进行数据修改，如果出现错误或者执行了rollback，系统可以利用undo log中备份将数据恢复到事务开始之前的状态。

# redo log

重做日志可以确保事务的持久性，防止在发生故障的时候，脏页没有刷盘。重启MySQL服务的时候可以根据redo log进行重做，从而达到事务持久性这一特性。

事务开始之后就会产生redo log，事务执行过程中就会逐步将redo log落盘。当事务对应的脏页写到磁盘后，redo log就没用了。

redo log是物理格式的日志，记录的是物理数据页的修改信息，redo log是顺序写入redo log file的物理文件中去的。

redo log会先写到Innodb_log_buffer缓冲区，然后通过以下方式再将缓冲区日志刷新到磁盘上：

- Master Thread每秒执行刷新Innodb_log_buffer到redo log 文件中
- 每个事务提交时会将redo log刷新到redo log文件中
- 当重做日志缓存可用空间少于一半时，重做日志缓存被刷新到redo log文件中

redo log记录的是新数据的备份，事务提交之前只要将redo log持久化即可，不需要持久化数据。系统崩溃时虽然数据没有持久化，但是redo log已经持久化，可以根据redo log内容将所有数据恢复到最新状态。

# 参考

- [https://www.cnblogs.com/wy123/p/8365234.html](https://www.cnblogs.com/wy123/p/8365234.html)
- [https://yq.aliyun.com/articles/592937](https://yq.aliyun.com/articles/592937)