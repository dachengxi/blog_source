---
title: JVM中的MinorGC-MajorGC-FullGC
date: 2017-06-17 16:02:05
categories: 
	- JVM
tags:
	- JVM
---

Minor GC、Major GC、Full GC

<!--more-->

HotSpot VM实现，GC分为两种：

- Partial GC（局部GC），不收集整个GC堆的模式
  - Young GC（Minor GC），只收集新生代的GC。
  - Old GC，只收集老年代的GC，只有CMS的并发收集是这个模式。
  - Mixed GC，收集整个新生代以及部分老年代的GC，是有G1有这个模式。
- Full GC，收集整个堆，包括新生代、老年代、永久代所有部分的模式。

Major GC通常是跟Full GC是等价的，收集整个GC堆。但因为HotSpot VM发展了这么多年，外界对各种名词的解读已经完全混乱了，当有人说“Major GC”的时候一定要问清楚他想要指的是上面的Full GC还是Old GC。

# Young GC（Minor GC）

发生在新生代的GC，当新生代中的Eden区满的时候触发。Young GC中有部分存活对象会晋升到老年代，所以Young GC后老年代的占用量通常会有所升高。

新生代分为Eden区和两个Survivor区，发生YGC的时候，Eden区和一个Survivor区中存活的对象被复制到另外一个空闲Survivor区。

## 晋升到老年代的条件

- 新生代的Eden区满的时候，会进行一次YGC，当Eden和Survivor区中依然存活的对象无法放到Survivor区中，通过分配担保机制提前转移到老年代中。这称为过早提升(Premature Promotion)，这会导致老年代中短期存活对象的增长，可能会引发严重的性能问题。在Minor GC过程中，如果老年代满了而无法容纳更多的对象，YGC 之后通常就会进行Full GC,这将导致遍历整个Java堆，这称为提升失败(Promotion Failure)。
- 新建对象太大，新生代无法容纳这个对象，会直接在老年代分配，参数`-XX:PretenureSizeThreshold`用来设置这个门限值，此参数只对Serial及ParNew两款收集器有效。
- 长期存活的对象进入老年代，对象头的Mark Word中包含对象的年龄，年龄达到了参数`-XX:MaxTenuringThreshold`指定的阈值（默认15，Mark Word中记录年轻的字段有4位），会晋升到老年代中。每一YGC后对象还能存活的话，年龄就会加1。
- 动态年龄对象判定，虚拟机不总是要求对象年龄必须达到`-XX:MaxTenuringThreshold`设置的值后才能晋升到老年代，如果在Survivor区中相同年龄的对象的所有大小之和超过Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，不需要等到参数`-XX:MaxTenuringThreshold`设置的阈值。

## Young GC和卡表

YGC存在一个问题，老年代对象可能引用新生代对象，在标记存活对象的时候，就需要扫描老年代的对象，如果该对象拥有对新生代对象的引用，那么这个引用也会被作为 GC Roots。这相当于就做了全堆扫描。为了避免全堆扫描，JVM使用了卡表技术：

![JVM卡表](./JVM中的MinorGC-MajorGC-FullGC/JVM卡表.png)

卡表的具体策略是：将老年代的空间分成大小为 512B的若干张卡，并且维护一个卡表，卡表本省是字节数组，数组中的每个元素对应着一张卡，其实就是一个标识位，这个标识位代表对应的卡是否可能存有指向新生代对象的引用，如果可能存在，那么我们认为这张卡是脏的，即脏卡。如上图所示，卡表3被标记为脏。

在进行YGC的时候，我们便可以不用扫描整个老年代，而是在卡表中寻找脏卡，并将脏卡中的老年代指向新生代的引用加入到 Minor GC的GC Roots里，当完成所有脏卡的扫描之后，Java 虚拟机便会将所有脏卡的标识位清零。这样虚拟机以空间换时间，避免了全表扫描。

## 空间分配担保

YGC之前，JVM会先检查老年代最大可用连续空间大小是否大于新生代所有对象的总大小，如果大于新生代所有对象总大小，这次YGC就是安全的。

如果小于新生代所有对象总大小，JVM会查看HandlePromotionFailure设置的值，如果设置为false表示不允许失败担保，则进行一次Full GC；如果设置为true表示允许担保失败，JVM会检查老年大最大可用连续空间大小是否大于历次晋升到老年代的对象的平均大小，如果大于的话则可以继续进行一次YGC，但是这次YGC是有风险的；如果小于的话，则进行一次Full GC。

# Full GC

针对不同的垃圾收集器，Full GC的触发条件可能不都一样。按HotSpot VM的serial GC的实现来看，触发条件是：

- 当准备要触发一次Young GC时，如果发现统计数据说之前Young GC的平均晋升大小比目前老年代剩余的空间大，则不会触发Young GC而是转为触发Full GC（因为HotSpot VM的GC里，除了CMS的并发收集之外，其它能收集老年代的GC都会同时收集整个GC堆，包括新生代，所以不需要事先触发一次单独的Young GC）。
- 如果有永久代（方法区）的话，要在永久代分配空间但已经没有足够空间时，也要触发一次Full GC。
- System.gc()、heap dump带GC，默认也是触发Full GC。
- 老年代空间不足，新生代对象转入以及创建大对象大数组时，空间不足，会导致Full GC；或者空间足够，但是没有足够的连续空间存放对象，会导致Full GC。CMS垃圾收集器提供了一个可配置的参数`-XX:+UseCMSCompactAtFullCollection`，用于在Full GC后进行碎片整理的，内存整理的过程无法并发的，需要stop the world，JVM还提供了另外一个参数`-XX:CMSFullGCsBeforeCompaction`用于设置在执行多少次不压缩的Full GC后，跟着来一次带压缩的。
- CMS GC出现promotion failed和concurrent mode failure会导致Full GC，promotion failed是在进行YGC时，survivor区放不下、对象只能放入老年代，而此时老年代也放不下造成的；concurrent mode failure是在执行CMS GC的过程中同时有对象要放入老年代，而此时老年代空间不足造成的（有时候空间不足是CMS GC时当前的浮动垃圾过多导致暂时性的空间不足触发Full GC）。


而在 Parallel Scavenge 收集器下，默认是在要触发 Full GC前先执行一次 YGC，并且两次GC之间能让应用程序稍微运行一小下，以期降低 Full GC的暂停时间 (因为 YGC 会尽量清理了新生代的死对象，减少了 Full GC的工作量)，控制这个行为的VM参数是`-XX:+ScavengeBeforeFullGC`。

并发GC的触发条件就不一样，以 CMS GC为例，它主要是定时去检查老年代的使用量，但使用量超过了触发比例就会启动一次 CMS GC，对老年代做并发收集。

# 参考

- **引用自R大（加粗加大这一行）：https://www.zhihu.com/question/41922036/answer/93079526**