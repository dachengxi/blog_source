---
title: Java中的ScheduledThreadPoolExecutor
date: 2016-05-30 17:57:06
categories: 
	- Java
tags:
	- Java
---

ScheduledThreadPoolExecutor使用和原理学习。

<!-- more -->

# 原理

ScheduledThreadPoolExecutor提供了延迟和周期执行功能，内部实现相当于使用DelayQueue延时队列、PriorityQueue优先级队列和Delayed三者来配合实现的功能，在使用延时队列实现我们自己的业务功能时，也会使用这三个组件来配合。但是实际上ScheduledThreadPoolExecutor不是直接使用这三个组件来实现，而是在内部实现了几个功能类似的内部类：

- DelayedWorkQueue，功能相当于DelayQueue，只不过这个内部没有依赖PriorityQueue，而是直接实现了堆排序功能。
- ScheduledFutureTask，相当于实现了Delayed接口的业务类，ScheduledFutureTask的父接口的父接口ScheduledFuture继承了Delayed接口，ScheduledFutureTask实现了getDelay和compareTo方法，这两个方法可以参考DelayQueue解析中的使用。



# 方法

- schedule，给定延迟后，执行任务
- scheduleAtFixedRate，每个任务都在指定的时间间隔内执行，如果一个任务瞬间执行完，指定的时间间隔还有很多剩余的，下一个任务也不会执行；如果一个任务在指定的时间间隔没有执行完，占用了下个任务的时间，那这个任务执行完后下个任务立马就开始。
- scheduleWithFixedDelay，在一次任务结束后，间隔指定的时间，再继续执行下一次任务，不管一次任务执行多长时间，在这次任务结束后都会暂停指定的时间，接下来再执行下面的任务。就是说我不管，我每次任务完成都必须要休息一定时间。

# 源码

