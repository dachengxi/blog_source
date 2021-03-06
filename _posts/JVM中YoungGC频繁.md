---
title: JVM中YoungGC频繁
date: 2017-06-24 23:47:36
categories: 
	- JVM
tags:
	- JVM
---
分析一下YGC频繁的原因。

<!--more-->

# GC log分析

## log中打印的参数

```
Java HotSpot(TM) 64-Bit Server VM (25.172-b11) for linux-amd64 JRE (1.8.0_172-b11), built on Mar 28 2018 21:44:09 by "java_re" with gcc 4.3.0 2008042
8 (Red Hat 4.3.0-8)
Memory: 4k page, physical 8028236k(5004928k free), swap 4096572k(4096572k free)
CommandLine flags: -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSScavengeBeforeRemark -XX:CompressedClassSpaceSize=528482304 -XX:+DisableExplicitGC -
XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/admin/service_heap_dump_20190409134805.hprof -XX:InitialHeapSize=4294967296 -XX:MaxHeapSize=429
4967296 -XX:MaxMetaspaceSize=536870912 -XX:MaxNewSize=2147483648 -XX:NewSize=2147483648 -XX:OldPLABSize=16 -XX:-OmitStackTraceInFastThrow -XX:+PrintG
C -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+ScavengeBeforeFullGC -XX:SurvivorRatio=8 -XX:+UseCompressedClassPointers -XX
:+UseCompressedOops -XX:+UseConcMarkSweepGC -XX:+UseParNewGC
```

- Memory: 4k page
- physical 8028236k(5004928k free)，物理内存，7.6G（4.77G空闲）
- swap 4096572k(4096572k free)，交换空间，4G（4G空闲）
- -XX:CMSInitiatingOccupancyFraction=70，CMS垃圾收集器，当老年代达到指定阈值时，触发CMS垃圾回收，这里是70%
- -XX:+CMSScavengeBeforeRemark，CMS在remark之前进行一次YGC，由于新生代存在引用老年代对象的情况，因此CMS remark阶段会将新生代作为老年代的GC ROOTS进行扫描，防止回收了不该回收的对象。而配置CMSScavengeBeforeRemark参数，在CMS GC的remark阶段开始前先进行一次YGC，有利于减少新生代对老年代的无效引用，降低remark阶段的时间开销。
- -XX:CompressedClassSpaceSize=528482304，设置Klass Metaspace的大小，500M
- -XX:+DisableExplicitGC，禁止代码中显示调用GC
- -XX:+HeapDumpOnOutOfMemoryError，当JVM发生OOM时，自动生成DUMP文件
- -XX:HeapDumpPath=/home/admin/service_heap_dump_20190409134805.hprof，生成DUMP文件的路径
- -XX:InitialHeapSize=4294967296，也就是-Xms指定的参数，初始堆内存大小，4G
- -XX:MaxHeapSize=4294967296，也就是-Xmx指定的参数，最大堆内存大小，4G
- -XX:MaxMetaspaceSize=536870912，Metaspace大小，512M
- -XX:MaxNewSize=2147483648，新生代可被分配的内存的最大上限，2G
- -XX:NewSize=2147483648，新生代初始化内存的大小，2G（-Xmn可同时对XX:NewSize和XX:MaxNewSize参数设置）
- -XX:OldPLABSize=16，老年代空间 PLAB 大小
- -XX:-OmitStackTraceInFastThrow，禁用Fast Throw优化
- -XX:+PrintGC，打印GC日志
- -XX:+PrintGCDateStamps，输出GC的时间戳
- -XX:+PrintGCDetails，打印GC的详细信息
- -XX:+PrintGCTimeStamps，输出GC时间戳
- -XX:+ScavengeBeforeFullGC，Full GC前执行一次YGC
- -XX:SurvivorRatio=8，Eden和两个Survivor的比例：8:1:1
- -XX:+UseCompressedClassPointers，压缩类指针
- -XX:+UseCompressedOops，压缩对象指针，oops指的是普通对象指针(ordinary object pointers)，Java堆中对象指针会被压缩成32位
- -XX:+UseConcMarkSweepGC，老年代收集器为CMS
- -XX:+UseParNewGC，新生代收集器为ParNew

## 实际启动参数

```
-server -Xms4g -Xmx4g -Xmn2g -XX:MaxMetaspaceSize=512m -XX:SurvivorRatio=8 -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=70 -XX:+ScavengeBeforeFullGC -XX:+CMSScavengeBeforeRemark -XX:+DisableExplicitGC -XX:-OmitStackTraceInFastThrow -XX:+PrintGCDateStamps -verbose:gc -XX:+PrintGCDetails -Xloggc:${HOME}/service_gc_`date +%Y%m%d%H%M%S`.log -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=${HOME}/service_heap_dump_`date +%Y%m%d%H%M%S`.hprof
```

- -Xms4g，初始堆大小，4G
- -Xmx4g，最大堆大小，4G
- -Xmn2g，新生代初始内存和最大内存都是2G

Eden=1.6g，Suvivor0=0.2g，Suvivor1=0.2g

## GC log

```
2019-04-10T20:04:51.945+0800: 109006.832: [GC (GCLocker Initiated GC) 2019-04-10T20:04:51.945+0800: 109006.832: [ParNew: 1823929K->209664K(1887488K),
 0.1568521 secs] 2587624K->1013834K(3984640K), 0.1572354 secs] [Times: user=0.58 sys=0.00, real=0.16 secs] 
2019-04-10T20:04:53.556+0800: 109008.442: [GC (Allocation Failure) 2019-04-10T20:04:53.556+0800: 109008.442: [ParNew: 1887488K->188658K(1887488K), 0.
0834678 secs] 2691658K->1039814K(3984640K), 0.0837781 secs] [Times: user=0.33 sys=0.00, real=0.08 secs] 
2019-04-10T20:04:54.928+0800: 109009.815: [GC (Allocation Failure) 2019-04-10T20:04:54.928+0800: 109009.815: [ParNew: 1866482K->147763K(1887488K), 0.
0521302 secs] 2717638K->999285K(3984640K), 0.0524123 secs] [Times: user=0.20 sys=0.00, real=0.05 secs] 
2019-04-10T20:04:56.314+0800: 109011.200: [GC (Allocation Failure) 2019-04-10T20:04:56.314+0800: 109011.201: [ParNew: 1825587K->164545K(1887488K), 0.
1519752 secs] 2677109K->1084741K(3984640K), 0.1522695 secs] [Times: user=0.56 sys=0.00, real=0.15 secs] 
2019-04-10T20:04:57.954+0800: 109012.841: [GC (Allocation Failure) 2019-04-10T20:04:57.954+0800: 109012.841: [ParNew: 1842369K->194192K(1887488K), 0.
1914321 secs] 2762565K->1234446K(3984640K), 0.1916991 secs] [Times: user=0.70 sys=0.00, real=0.19 secs] 
2019-04-10T20:04:59.497+0800: 109014.384: [GC (Allocation Failure) 2019-04-10T20:04:59.497+0800: 109014.384: [ParNew: 1872016K->160741K(1887488K), 0.
0486877 secs] 2912270K->1201078K(3984640K), 0.0489626 secs] [Times: user=0.19 sys=0.00, real=0.05 secs] 
2019-04-10T20:05:00.812+0800: 109015.699: [GC (Allocation Failure) 2019-04-10T20:05:00.813+0800: 109015.699: [ParNew: 1835736K->151823K(1887488K), 0.
1376864 secs] 2876073K->1255445K(3984640K), 0.1380070 secs] [Times: user=0.51 sys=0.00, real=0.14 secs] 
2019-04-10T20:05:02.554+0800: 109017.440: [GC (Allocation Failure) 2019-04-10T20:05:02.554+0800: 109017.440: [ParNew: 1829647K->171511K(1887488K), 0.
1798695 secs] 2933269K->1387182K(3984640K), 0.1801425 secs] [Times: user=0.67 sys=0.00, real=0.18 secs] 
2019-04-10T20:05:04.059+0800: 109018.946: [GC (Allocation Failure) 2019-04-10T20:05:04.059+0800: 109018.946: [ParNew: 1849335K->167007K(1887488K), 0.
0483309 secs] 3065006K->1383074K(3984640K), 0.0486151 secs] [Times: user=0.16 sys=0.00, real=0.05 secs] 
2019-04-10T20:05:05.628+0800: 109020.515: [GC (Allocation Failure) 2019-04-10T20:05:05.628+0800: 109020.515: [ParNew: 1844831K->209664K(1887488K), 0.
1480378 secs] 3060898K->1457657K(3984640K), 0.1483184 secs] [Times: user=0.57 sys=0.00, real=0.15 secs] 
2019-04-10T20:05:07.079+0800: 109021.966: [GC (Allocation Failure) 2019-04-10T20:05:07.079+0800: 109021.966: [ParNew: 1887488K->170162K(1887488K), 0.
0579737 secs] 3135481K->1418544K(3984640K), 0.0582560 secs] [Times: user=0.23 sys=0.00, real=0.06 secs] 
2019-04-10T20:05:08.604+0800: 109023.491: [GC (Allocation Failure) 2019-04-10T20:05:08.604+0800: 109023.491: [ParNew: 1847986K->209664K(1887488K), 0.
1702964 secs] 3096368K->1587027K(3984640K), 0.1705767 secs] [Times: user=0.63 sys=0.00, real=0.18 secs] 
2019-04-10T20:05:10.186+0800: 109025.073: [GC (Allocation Failure) 2019-04-10T20:05:10.186+0800: 109025.073: [ParNew: 1887488K->200295K(1887488K), 0.
1819795 secs] 3264851K->1676149K(3984640K), 0.1822649 secs] [Times: user=0.67 sys=0.00, real=0.18 secs] 
2019-04-10T20:05:10.372+0800: 109025.259: [GC (CMS Initial Mark) [1 CMS-initial-mark: 1475854K(2097152K)] 1694195K(3984640K), 0.0302480 secs] [Times:
 user=0.07 sys=0.00, real=0.03 secs] 
2019-04-10T20:05:10.403+0800: 109025.289: [CMS-concurrent-mark-start]
2019-04-10T20:05:10.565+0800: 109025.452: [CMS-concurrent-mark: 0.162/0.162 secs] [Times: user=0.36 sys=0.00, real=0.17 secs] 
2019-04-10T20:05:10.565+0800: 109025.452: [CMS-concurrent-preclean-start]
2019-04-10T20:05:10.571+0800: 109025.458: [CMS-concurrent-preclean: 0.006/0.006 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
2019-04-10T20:05:10.571+0800: 109025.458: [CMS-concurrent-abortable-preclean-start]
2019-04-10T20:05:11.842+0800: 109026.729: [GC (GCLocker Initiated GC) 2019-04-10T20:05:11.842+0800: 109026.729: [ParNew: 1849355K->187949K(1887488K),
 0.1785371 secs] 3357977K->1840092K(3984640K), 0.1789082 secs] [Times: user=0.65 sys=0.00, real=0.18 secs] 
2019-04-10T20:05:13.012+0800: 109027.899: [CMS-concurrent-abortable-preclean: 2.251/2.441 secs] [Times: user=6.67 sys=0.00, real=2.44 secs] 
2019-04-10T20:05:13.013+0800: 109027.900: [GC (CMS Final Remark) [YG occupancy: 1220076 K (1887488 K)]2019-04-10T20:05:13.013+0800: 109027.900: [GC (
CMS Final Remark) 2019-04-10T20:05:13.014+0800: 109027.900: [ParNew: 1220076K->154513K(1887488K), 0.1657934 secs] 2872219K->1924261K(3984640K), 0.166
0497 secs] [Times: user=0.60 sys=0.00, real=0.17 secs] 
2019-04-10T20:05:13.180+0800: 109028.066: [Rescan (parallel) , 0.0454351 secs]2019-04-10T20:05:13.225+0800: 109028.112: [weak refs processing, 0.0001
084 secs]2019-04-10T20:05:13.225+0800: 109028.112: [class unloading, 0.0520415 secs]2019-04-10T20:05:13.277+0800: 109028.164: [scrub symbol table, 0.
0156959 secs]2019-04-10T20:05:13.293+0800: 109028.180: [scrub string table, 0.0014593 secs][1 CMS-remark: 1769747K(2097152K)] 1924261K(3984640K), 0.3
026953 secs] [Times: user=0.87 sys=0.00, real=0.31 secs] 
2019-04-10T20:05:13.316+0800: 109028.203: [CMS-concurrent-sweep-start]
2019-04-10T20:05:14.813+0800: 109029.700: [GC (Allocation Failure) 2019-04-10T20:05:14.813+0800: 109029.700: [ParNew: 1832337K->209664K(1887488K), 0.
1690814 secs] 2401415K->829430K(3984640K), 0.1694105 secs] [Times: user=0.64 sys=0.00, real=0.17 secs] 
2019-04-10T20:05:15.076+0800: 109029.963: [CMS-concurrent-sweep: 1.586/1.760 secs] [Times: user=5.11 sys=0.00, real=1.76 secs] 
2019-04-10T20:05:15.076+0800: 109029.963: [CMS-concurrent-reset-start]
2019-04-10T20:05:15.080+0800: 109029.967: [CMS-concurrent-reset: 0.004/0.004 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
2019-04-10T20:05:16.566+0800: 109031.452: [GC (Allocation Failure) 2019-04-10T20:05:16.566+0800: 109031.452: [ParNew: 1887488K->209664K(1887488K), 0.
0852347 secs] 2376299K->751020K(3984640K), 0.0855516 secs] [Times: user=0.34 sys=0.00, real=0.08 secs] 
2019-04-10T20:05:16.655+0800: 109031.541: [GC (GCLocker Initiated GC) 2019-04-10T20:05:16.655+0800: 109031.542: [ParNew: 226066K->69094K(1887488K), 0
.1444585 secs] 767423K->723085K(3984640K), 0.1446827 secs] [Times: user=0.42 sys=0.00, real=0.15 secs] 
2019-04-10T20:05:18.192+0800: 109033.079: [GC (Allocation Failure) 2019-04-10T20:05:18.193+0800: 109033.079: [ParNew: 1746918K->121493K(1887488K), 0.
0520770 secs] 2400909K->775483K(3984640K), 0.0523791 secs] [Times: user=0.21 sys=0.00, real=0.05 secs] 
2019-04-10T20:05:19.766+0800: 109034.653: [GC (Allocation Failure) 2019-04-10T20:05:19.766+0800: 109034.653: [ParNew: 1799317K->172147K(1887488K), 0.
2003248 secs] 2453307K->940006K(3984640K), 0.2009707 secs] [Times: user=0.71 sys=0.00, real=0.20 secs] 
2019-04-10T20:05:21.337+0800: 109036.224: [GC (Allocation Failure) 2019-04-10T20:05:21.337+0800: 109036.224: [ParNew: 1850818K->173032K(1887488K), 0.
2056822 secs] 2618676K->1080561K(3984640K), 0.2060002 secs] [Times: user=0.72 sys=0.00, real=0.20 secs] 
2019-04-10T20:05:21.546+0800: 109036.433: [GC (GCLocker Initiated GC) 2019-04-10T20:05:21.547+0800: 109036.433: [ParNew: 190483K->57139K(1887488K), 0
.0843375 secs] 1098011K->1080787K(3984640K), 0.0845775 secs] [Times: user=0.29 sys=0.00, real=0.08 secs] 
2019-04-10T20:05:23.036+0800: 109037.923: [GC (Allocation Failure) 2019-04-10T20:05:23.036+0800: 109037.923: [ParNew: 1734963K->142250K(1887488K), 0.
0654108 secs] 2758611K->1165898K(3984640K), 0.0657093 secs] [Times: user=0.25 sys=0.00, real=0.07 secs] 
2019-04-10T20:05:24.549+0800: 109039.435: [GC (GCLocker Initiated GC) 2019-04-10T20:05:24.549+0800: 109039.436: [ParNew: 1820074K->168253K(1887488K),
 0.1858657 secs] 2843722K->1321273K(3984640K), 0.1861676 secs] [Times: user=0.68 sys=0.00, real=0.18 secs] 
2019-04-10T20:05:26.085+0800: 109040.972: [GC (Allocation Failure) 2019-04-10T20:05:26.085+0800: 109040.972: [ParNew: 1846077K->154005K(1887488K), 0.
0462496 secs] 2999097K->1307234K(3984640K), 0.0465630 secs] [Times: user=0.18 sys=0.00, real=0.05 secs] 
2019-04-10T20:05:27.460+0800: 109042.347: [GC (GCLocker Initiated GC) 2019-04-10T20:05:27.460+0800: 109042.347: [ParNew: 1857495K->209664K(1887488K),
 0.0913403 secs] 3010724K->1371383K(3984640K), 0.0916332 secs] [Times: user=0.36 sys=0.00, real=0.09 secs] 
2019-04-10T20:05:28.944+0800: 109043.831: [GC (GCLocker Initiated GC) 2019-04-10T20:05:28.944+0800: 109043.831: [ParNew: 1887488K->204384K(1887488K),
 0.1025569 secs] 3049209K->1406621K(3984640K), 0.1028616 secs] [Times: user=0.39 sys=0.00, real=0.11 secs] 
2019-04-10T20:05:30.291+0800: 109045.178: [GC (Allocation Failure) 2019-04-10T20:05:30.291+0800: 109045.178: [ParNew: 1882208K->171428K(1887488K), 0.
0444964 secs] 3084445K->1373752K(3984640K), 0.0448216 secs] [Times: user=0.14 sys=0.00, real=0.04 secs] 
2019-04-10T20:05:31.769+0800: 109046.656: [GC (Allocation Failure) 2019-04-10T20:05:31.769+0800: 109046.656: [ParNew: 1840540K->209664K(1887488K), 0.
1070460 secs] 3042865K->1414779K(3984640K), 0.1073485 secs] [Times: user=0.41 sys=0.00, real=0.10 secs] 
2019-04-10T20:05:33.293+0800: 109048.180: [GC (Allocation Failure) 2019-04-10T20:05:33.293+0800: 109048.180: [ParNew: 1887488K->174620K(1887488K), 0.
1014037 secs] 3092603K->1426287K(3984640K), 0.1017165 secs] [Times: user=0.39 sys=0.00, real=0.10 secs] 
2019-04-10T20:05:34.713+0800: 109049.600: [GC (Allocation Failure) 2019-04-10T20:05:34.713+0800: 109049.600: [ParNew: 1852444K->149718K(1887488K), 0.
0388156 secs] 3104111K->1401511K(3984640K), 0.0391062 secs] [Times: user=0.13 sys=0.00, real=0.04 secs] 
2019-04-10T20:05:36.300+0800: 109051.187: [GC (Allocation Failure) 2019-04-10T20:05:36.300+0800: 109051.187: [ParNew: 1827542K->209664K(1887488K), 0.
1370734 secs] 3079335K->1482894K(3984640K), 0.1373627 secs] [Times: user=0.52 sys=0.00, real=0.13 secs] 
2019-04-10T20:05:37.822+0800: 109052.709: [GC (Allocation Failure) 2019-04-10T20:05:37.823+0800: 109052.709: [ParNew: 1887488K->168568K(1887488K), 0.
0630131 secs] 3160718K->1442889K(3984640K), 0.0633086 secs] [Times: user=0.25 sys=0.00, real=0.06 secs] 
2019-04-10T20:05:39.229+0800: 109054.116: [GC (GCLocker Initiated GC) 2019-04-10T20:05:39.229+0800: 109054.116: [ParNew: 1862776K->199264K(1887488K),
 0.1551475 secs] 3137097K->1600406K(3984640K), 0.1554344 secs] [Times: user=0.57 sys=0.00, real=0.15 secs] 
2019-04-10T20:05:40.850+0800: 109055.736: [GC (Allocation Failure) 2019-04-10T20:05:40.850+0800: 109055.737: [ParNew: 1877088K->165034K(1887488K), 0.
1900999 secs] 3278230K->1682137K(3984640K), 0.1904167 secs] [Times: user=0.69 sys=0.00, real=0.19 secs] 
2019-04-10T20:05:41.041+0800: 109055.928: [GC (CMS Initial Mark) [1 CMS-initial-mark: 1517102K(2097152K)] 1698563K(3984640K), 0.0451473 secs] [Times:
 user=0.11 sys=0.00, real=0.04 secs]
```
## ParNew日志

```
2019-04-10T20:04:53.556+0800: 109008.442: [GC (Allocation Failure) 2019-04-10T20:04:53.556+0800: 109008.442: [ParNew: 1887488K->188658K(1887488K), 0.0834678 secs] 2691658K->1039814K(3984640K), 0.0837781 secs] [Times: user=0.33 sys=0.00, real=0.08 secs] 
```

- GC，表示是一次YGC
- Allocation Failure，表示失败类型
- ParNew，收集器名称，ParNew收集器，多线程，并行，Stop the World
- 1887488K->188658K，收集前后新生代情况，1.8G -> 184M
- (1887488K)，新生代容量，1.8G
- 0.0834678 secs，新生代GC花费的时间
- 2691658K->1039814K，收集前后整个堆的使用情况，2.5G -> 0.99G
- (3984640K)，整个堆的容量，3.8G
- 0.0837781 secs，GC总时间，ParNew 收集器标记和复制新生代活着的对象所花费的时间（包括和老年代通信的开销、对象晋升到老年代开销、垃圾收集周期结束一些最后的清理对象等的花销）
- Times: user=0.33 sys=0.00, real=0.08 secs
  - user，GC线程在垃圾收集期间使用的CPU总时间
  - sys，系统调用或者等待系统事件花费的时间
  - real，应用被暂停的时间，GC 线程是多线程的，导致了 real 小于 (user+sys)，如果是 gc 线程是单线程的话，real 是接近于 (user+sys) 时间

收集前新生代大小1887488K->收集后新生代大小188658K(新生代总大小1887488K)，YGC后新生代大小减少了1698830K；收集前堆大小2691658K->收集后堆大小1039814K(堆总大小3984640K)，YGC后新生代大小减少了1651844K，晋升到老年代的大小为：1698830K - 1651844K = 46986K

## CMS日志

```
2019-04-10T20:05:10.372+0800: 109025.259: [GC (CMS Initial Mark) [1 CMS-initial-mark: 1475854K(2097152K)] 1694195K(3984640K), 0.0302480 secs] [Times:
 user=0.07 sys=0.00, real=0.03 secs] 
2019-04-10T20:05:10.403+0800: 109025.289: [CMS-concurrent-mark-start]
2019-04-10T20:05:10.565+0800: 109025.452: [CMS-concurrent-mark: 0.162/0.162 secs] [Times: user=0.36 sys=0.00, real=0.17 secs] 
2019-04-10T20:05:10.565+0800: 109025.452: [CMS-concurrent-preclean-start]
2019-04-10T20:05:10.571+0800: 109025.458: [CMS-concurrent-preclean: 0.006/0.006 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
2019-04-10T20:05:10.571+0800: 109025.458: [CMS-concurrent-abortable-preclean-start]
2019-04-10T20:05:11.842+0800: 109026.729: [GC (GCLocker Initiated GC) 2019-04-10T20:05:11.842+0800: 109026.729: [ParNew: 1849355K->187949K(1887488K),
 0.1785371 secs] 3357977K->1840092K(3984640K), 0.1789082 secs] [Times: user=0.65 sys=0.00, real=0.18 secs] 
2019-04-10T20:05:13.012+0800: 109027.899: [CMS-concurrent-abortable-preclean: 2.251/2.441 secs] [Times: user=6.67 sys=0.00, real=2.44 secs] 
2019-04-10T20:05:13.013+0800: 109027.900: [GC (CMS Final Remark) [YG occupancy: 1220076 K (1887488 K)]2019-04-10T20:05:13.013+0800: 109027.900: [GC (
CMS Final Remark) 2019-04-10T20:05:13.014+0800: 109027.900: [ParNew: 1220076K->154513K(1887488K), 0.1657934 secs] 2872219K->1924261K(3984640K), 0.166
0497 secs] [Times: user=0.60 sys=0.00, real=0.17 secs] 
2019-04-10T20:05:13.180+0800: 109028.066: [Rescan (parallel) , 0.0454351 secs]2019-04-10T20:05:13.225+0800: 109028.112: [weak refs processing, 0.0001
084 secs]2019-04-10T20:05:13.225+0800: 109028.112: [class unloading, 0.0520415 secs]2019-04-10T20:05:13.277+0800: 109028.164: [scrub symbol table, 0.
0156959 secs]2019-04-10T20:05:13.293+0800: 109028.180: [scrub string table, 0.0014593 secs][1 CMS-remark: 1769747K(2097152K)] 1924261K(3984640K), 0.3
026953 secs] [Times: user=0.87 sys=0.00, real=0.31 secs] 
2019-04-10T20:05:13.316+0800: 109028.203: [CMS-concurrent-sweep-start]
2019-04-10T20:05:14.813+0800: 109029.700: [GC (Allocation Failure) 2019-04-10T20:05:14.813+0800: 109029.700: [ParNew: 1832337K->209664K(1887488K), 0.
1690814 secs] 2401415K->829430K(3984640K), 0.1694105 secs] [Times: user=0.64 sys=0.00, real=0.17 secs] 
2019-04-10T20:05:15.076+0800: 109029.963: [CMS-concurrent-sweep: 1.586/1.760 secs] [Times: user=5.11 sys=0.00, real=1.76 secs] 
2019-04-10T20:05:15.076+0800: 109029.963: [CMS-concurrent-reset-start]
2019-04-10T20:05:15.080+0800: 109029.967: [CMS-concurrent-reset: 0.004/0.004 secs] [Times: user=0.01 sys=0.00, real=0.00 secs]
```

### 初始标记阶段

```
2019-04-10T20:05:10.372+0800: 109025.259: [GC (CMS Initial Mark) [1 CMS-initial-mark: 1475854K(2097152K)] 1694195K(3984640K), 0.0302480 secs] [Times:
 user=0.07 sys=0.00, real=0.03 secs] 
```

- GC (CMS Initial Mark)，CMS收集器的初始标记阶段
- 1475854K(2097152K)，老年代占用1475854K（1.4G），老年代容量2097152K（2G）
- 1694195K(3984640K)，整个堆的占用1694195K（1.6G），整个堆的容量3984640K（3.8G）
- 0.0302480 secs，本次标记耗时

该阶段从垃圾回收的根对象开始，且只扫描直接与根对象直接关联的对象，并做标记，需要Stop The Word，速度很快。

### 并发标记

```
2019-04-10T20:05:10.403+0800: 109025.289: [CMS-concurrent-mark-start]
2019-04-10T20:05:10.565+0800: 109025.452: [CMS-concurrent-mark: 0.162/0.162 secs] [Times: user=0.36 sys=0.00, real=0.17 secs] 
2019-04-10T20:05:10.565+0800: 109025.452: [CMS-concurrent-preclean-start]
2019-04-10T20:05:10.571+0800: 109025.458: [CMS-concurrent-preclean: 0.006/0.006 secs] [Times: user=0.01 sys=0.00, real=0.00 secs]
2019-04-10T20:05:10.571+0800: 109025.458: [CMS-concurrent-abortable-preclean-start]
2019-04-10T20:05:11.842+0800: 109026.729: [GC (GCLocker Initiated GC) 2019-04-10T20:05:11.842+0800: 109026.729: [ParNew: 1849355K->187949K(1887488K),
 0.1785371 secs] 3357977K->1840092K(3984640K), 0.1789082 secs] [Times: user=0.65 sys=0.00, real=0.18 secs] 
2019-04-10T20:05:13.012+0800: 109027.899: [CMS-concurrent-abortable-preclean: 2.251/2.441 secs] [Times: user=6.67 sys=0.00, real=2.44 secs] 
```

- CMS-concurrent-mark-start，表示并发标记阶段开始，会遍历整个年老代并且标记活着的对象
- CMS-concurrent-mark: 0.162/0.162 secs表示该阶段持续的时间和时钟时间，耗时0.162秒，耗时稍长。由于该阶运行的过程中用户线程也在运行，这就可能会发生这样的情况，已经被遍历过的对象的引用被用户线程改变，如果发生了这样的情况，JVM就会标记这个区域为Dirty Card。
- CMS-concurrent-preclean-start，预清理开始，该阶段检查并发标记阶段时从新生代晋升的对象，或新分配的对象，或被应用程序线程更新过的对象，（也就是上一个阶段被标记为Dirty Card的对象），帮助减少重新标记阶段的暂停时间。如果在此之后，Eden的占用量>CMSScheduleRemarkEdenSizeThreshold(默认为2M),会启动CMS-concurrent-abortable-preclean，预清理阶段只是一个取样过程，它将新生代按一定间隔进行分块，标记起始位置，以便remark时可以并行的的对块进行trace。不必从开头一点一点进行trace，预清理阶段时，最好发生一次ygc
- CMS-concurrent-preclean: 0.006/0.006 secs，预清理耗时
- CMS-concurrent-abortable-preclean-start，并发预清理开始，继续预清理，至到Eden区占用量达到CMSScheduleRemarkEdenPenetration(默认50%)，或达到5秒钟。但是如果ygc在这个阶段中没有发生的话，是达不到理想效果的。此时可以指定CMSMaxAbortablePrecleanTime，但是，等待一般都不是什么好的策略，可以采用CMSScavengeBeforeRemark，使remark之前发生一次YGC，从而减少remark阶段暂停的时间。
- CMS-concurrent-abortable-preclean: 2.251/2.441 secs，并发预清理耗时

### 重新标记阶段

```
2019-04-10T20:05:13.013+0800: 109027.900: [GC (CMS Final Remark) [YG occupancy: 1220076 K (1887488 K)]2019-04-10T20:05:13.013+0800: 109027.900: [GC (
CMS Final Remark) 2019-04-10T20:05:13.014+0800: 109027.900: [ParNew: 1220076K->154513K(1887488K), 0.1657934 secs] 2872219K->1924261K(3984640K), 0.166
0497 secs] [Times: user=0.60 sys=0.00, real=0.17 secs] 
2019-04-10T20:05:13.180+0800: 109028.066: [Rescan (parallel) , 0.0454351 secs]2019-04-10T20:05:13.225+0800: 109028.112: [weak refs processing, 0.0001
084 secs]2019-04-10T20:05:13.225+0800: 109028.112: [class unloading, 0.0520415 secs]2019-04-10T20:05:13.277+0800: 109028.164: [scrub symbol table, 0.
0156959 secs]2019-04-10T20:05:13.293+0800: 109028.180: [scrub string table, 0.0014593 secs][1 CMS-remark: 1769747K(2097152K)] 1924261K(3984640K), 0.3
026953 secs] [Times: user=0.87 sys=0.00, real=0.31 secs] 
```

- [GC (CMS Final Remark)，CMS重新标记阶段，该阶段的任务是完成标记整个年老代的所有的存活对象，尽管先前的pre clean阶段尽量应对处理了并发运行时用户线程改变的对象应用的标记，但是不可能跟上对象改变的速度，只是为final remark阶段尽量减少了负担。会Stop The World。
- YG occupancy: 1220076 K (1887488 K)，新生代当前内存占用情况，占用1220076k（1.1G），容量1887488K（1.8G），通常Final Remark阶段要尽量运行在年轻代是足够干净的时候，这样可以消除紧接着的连续的几个STW阶段。
- Rescan (parallel) 0.0454351 secs，整个final remark阶段扫描对象的用时总计，该阶段会重新扫描CMS堆中剩余的对象，重新从“根对象”开始扫描，并且也会处理对象关联。本次扫描共耗时0.0454351秒。
- weak refs processing, 0.0001084 secs，对弱引用处理的耗时
- class unloading, 0.0520415 secs，卸载无用类的耗时
- scrub symbol table, 0.0156959 secs，scrub string table, 0.0014593 secs，清理分别包含类级元数据和内部化字符串的符号和字符串表的耗时
- 1 CMS-remark: 1769747K(2097152K)，经历了上面阶段后老年代使用情况，老年代占用1769747K（1.6g），老年代容量2097152K(2G）
- 1924261K(3984640K), 0.3026953 secs，表示经历了final remark后整个堆的使用情况和final remark耗时，堆占用1924261K（1.8G），堆容量3984640K（3.8G）

### 并发清除阶段

```
2019-04-10T20:05:13.316+0800: 109028.203: [CMS-concurrent-sweep-start]
2019-04-10T20:05:14.813+0800: 109029.700: [GC (Allocation Failure) 2019-04-10T20:05:14.813+0800: 109029.700: [ParNew: 1832337K->209664K(1887488K), 0.
1690814 secs] 2401415K->829430K(3984640K), 0.1694105 secs] [Times: user=0.64 sys=0.00, real=0.17 secs] 
2019-04-10T20:05:15.076+0800: 109029.963: [CMS-concurrent-sweep: 1.586/1.760 secs] [Times: user=5.11 sys=0.00, real=1.76 secs] 
2019-04-10T20:05:15.076+0800: 109029.963: [CMS-concurrent-reset-start]
2019-04-10T20:05:15.080+0800: 109029.967: [CMS-concurrent-reset: 0.004/0.004 secs] [Times: user=0.01 sys=0.00, real=0.00 secs]
```

- CMS-concurrent-sweep-start，并发清除阶段开始，清除之前标记阶段没有被标记的无用对象并回收内存
- CMS-concurrent-sweep: 1.586/1.760 secs，清除没有标记的无用对象并回收内存的耗时
- CMS-concurrent-reset-start，重新设置CMS算法内部数据结构开始
- CMS-concurrent-reset: 0.004/0.004 secs，重置CMS内部算法数据结构耗时

# YGC频繁

报警提示YGC频繁，日志显示每秒一次YGC，单次耗时50ms - 200ms，一分钟多一次Major GC

参数：

堆大小：4G，新生代大小：2G，Eden：1.6G，Survivor0：200M，Suvivor1:200M

YGC触发原因：

- Eden区满的时候，会触发YGC

YGC频繁的原因：

- 产生了太多的短期对象，导致频繁的YGC
- 新生代的空间太小

这里场景是需要根据一个优惠券查询所有绑定了这个优惠券的商品，如果是商户优惠券或者类目优惠券，一个券将会有上万的商品关系，一次券的操作就会产生大量的对象并且都是一次性的。

调整：

堆大小增加到6G，新生代大小4G，-XX:SurvivorRatio=6，Eden：3.2G，Suvivor0：1.4g，Survivor1：1.4g

程序优化并且调整了参数后，没再发生过YGC频繁报警，Eden增大后并且Suvivor增大后，YGC次数下降，并且由于Suvivor增大，很多短期对象不再晋升到老年代，Major GC次数也减少了。