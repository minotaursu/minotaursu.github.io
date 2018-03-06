title: 所有需要知道的GC知识
date: 2018-03-03 00:24:51
tags:
- java
- gc
- 性能

---

![](http://hexo-tuchuan.qiniudn.com/jvmspeed.png)
作为一个java语言开发的应用，为了每一个0.1%的性能提升，折腾过了网络，IO，数据库之后难免要折腾一下GC。写下这篇日志的目的是希望看完这篇日志后，关于GC的问题能从google+stackoverflow寻找答案的过程中解脱出来，99%的GC问题都能独立解决，也方便自己温故知新。日志将从JVM内存结构 - 垃圾回收器 - 一些JVM参数 - 看懂GC日志 - 实际案例 - 建议配置 这几个方面详细介绍GC。在写这篇日志的过程中发现越写越长，需要详细介绍和深究原理的内容越来越多，不由得感慨网上的blog只能传递某几个知识点，想要构建一个完整的知识网络还是需要深入阅读相关的书籍，唯有将知识网络构建的越来越严密才能应对遇到的各种问题。
#JVM内存结构
目前主流的JVM都将内存分成不同的区域，JVM内存结构主要由堆，方法区和栈组成，堆作为最大一块的内存又由Eden空间、From Survivor空间、To Survivor空间，Old空间组成。
![](http://hexo-tuchuan.qiniudn.com/jvm.png)
新生代（Young Generation）：大多数对象在新生代中被创建(某些大对象可能直接创建在老年代)，其中很多对象的生命周期很短。
一般而言新生代内又分三个区：一个Eden区，两个Survivor区，当Eden区满时，还存活的对象将被复制到其中一个Survivor区。当这个Survivor区满时，此区的存活且不满足“晋升”条件的对象将被复制到另外一个Survivor区。对象每经历一次Minor GC，年龄加1，达到“晋升年龄阈值”后，被放到老年代，这个过程也称为“晋升”。在Serial和ParNew GC两种回收器中，“晋升年龄阈值”通过参数MaxTenuringThreshold设定，默认值为15。
老年代（Old Generation）：在新生代中经历了N次垃圾回收后仍然存活的对象，就会被放到年老代，该区域中对象存活率高。老年代的垃圾回收（又称Major GC）通常使用“标记-清理”或“标记-整理”算法。整堆包括新生代和老年代的垃圾回收称为Full GC。

#垃圾回收器
###串行（Serial）回收器
串行（Serial）回收器是单线程的一个回收器，简单、易实现、效率高。在串行收集器进行垃圾回收时，Java 应用程序中的线程都需要暂停，等待垃圾回收的完成，这样给用户体验造成较差效果。虽然如此，串行收集器却是一个成熟、经过长时间生产环境考验的极为高效的收集器。新生代串行处理器使用复制算法，实现相对简单，逻辑处理特别高效，且没有线程切换的开销。在诸如单 CPU 处理器或者较小的应用内存等硬件平台不是特别优越的场合，它的性能表现可以超过并行回收器和并发回收器。在 HotSpot 虚拟机中，使用-XX：+UseSerialGC 参数可以指定使用新生代串行收集器和老年代串行收集器。当 JVM 在 Client 模式下运行时，它是默认的垃圾收集器。
###并行（ParNew）回收器
并行（ParNew）回收器是Serial的多线程版，可以充分的利用CPU资源，减少回收的时间。并行回收器也是独占式的回收器，在收集过程中，应用程序会全部暂停。但由于并行回收器使用多线程进行垃圾回收，因此，在并发能力比较强的 CPU 上，它产生的停顿时间要短于串行回收器，而在单 CPU 或者并发能力较弱的系统中，并行回收器的效果不会比串行回收器好，由于多线程的压力，它的实际表现很可能比串行回收器差。
开启并行回收器可以使用参数-XX:+UseParNewGC，该参数设置新生代使用并行收集器，老年代使用串行收集器。
###吞吐量优先（Parallel Scavenge）回收器
吞吐量优先（Parallel Scavenge）回收器，侧重于吞吐量的控制。新生代并行回收收集器也是使用复制算法的收集器。从表面上看，它和并行收集器一样都是多线程、独占式的收集器。但是，并行回收收集器有一个重要的特点：它非常关注系统的吞吐量。使用-XX:+UseParallelOldGC 可以在新生代和老生代都使用并行回收收集器，这是一对非常关注吞吐量的垃圾收集器组合，在对吞吐量敏感的系统中，可以考虑使用。参数-XX:ParallelGCThreads也可以用于设置垃圾回收时的线程数量。
###并发标记清除（CMS，Concurrent Mark Sweep）回收器
并发标记清除（CMS，Concurrent Mark Sweep）回收器是一种以获取最短回收停顿时间为目标的回收器，该回收器是基于“标记-清除”算法实现的。与并行回收收集器不同，CMS 收集器主要关注于系统停顿时间。CMS 是 Concurrent Mark Sweep 的缩写，意为并发标记清除，从名称上可以得知，它使用的是标记-清除算法，同时它又是一个使用多线程并发回收的垃圾收集器。
CMS 工作时，主要步骤有：初始标记、并发标记、重新标记、并发清除和并发重置。其中初始标记和重新标记是独占系统资源的，而并发标记、并发清除和并发重置是可以和用户线程一起执行的。因此，从整体上来说，CMS 收集不是独占式的，它可以在应用程序运行过程中进行垃圾回收。
###G1回收器 (Garbage First)
G1 收集器的目标是作为一款服务器的垃圾收集器，因此，它在吞吐量和停顿控制上，预期要优于 CMS 收集器。与 CMS 收集器相比，G1收集器是基于标记-压缩算法的。因此，它不会产生空间碎片，也没有必要在收集完成后，进行一次独占式的碎片整理工作。G1 收集器还可以进行非常精确的停顿控制。它可以让开发人员指定当停顿时长为 M 时，垃圾回收时间不超过 N。使用参数-XX:+UnlockExperimentalVMOptions –XX:+UseG1GC 来启用 G1 回收器，设置 G1 回收器的目标停顿时间：-XX:MaxGCPauseMills=20,-XX:GCPauseIntervalMills=200。

![](http://hexo-tuchuan.qiniudn.com/garbage.png)

#一些参数
-Xms 设置堆的最小空间大小。
-Xmx 设置堆的最大空间大小。
-Xmn 设置新生代最小空间大小（推荐使用）。
-XX:NewSize 设置新生代最小空间大小。
-XX:MaxNewSize 设置新生代最大空间大小。
-XX:SurvivorRatio 设置Eden:Survivor中的一个比例,默认是8:1:1
-XX:PermSize 设置永久代最小空间大小。
-XX:MaxPermSize 设置永久代最大空间大小。
-Xss 设置每个线程的堆栈大小
老年代空间大小=堆空间大小-年轻代大空间大小
-XX:+DisableExplicitGC 显示禁止System.gc()，System.gc()变成一行无效的代码，如果系统有使用堆外内存不建议配置这个jvm参数
-XX:+UseConcMarkSweepGC 设置老年代为并发收集
-XX:+CMSParallelRemarkEnabled 降低标记停顿
-XX:+UseCMSCompactAtFullCollection 打开对年老代的压缩。可能会影响性能，但是可以消除碎片
-XX:LargePageSizeInBytes 设置内存页的大小
-XX+UseCMSInitiatingOccupancyOnly 标志来命令JVM不基于运行时收集的数据来启动CMS垃圾收集周期，始终使用CMSInitiatingPermOccupancyFraction的值（不推荐使用）
-XX:CMSInitiatingOccupancyFraction，默认为68，即当年老代的空间使用率达到68%时，会执行一次CMS回收。如果应用程序的内存使用率增长很快，在CMS的执行过程中，已经出现了内存不足，此时，CMS回收就会失败，虚拟机将启动SerialOld串行收集器进行垃圾回收。如果这样，应用程序将完全中断，直到垃圾回收完成，这时，应用程序的停顿时间可能会较长
-XX:+CMSScavengeBeforeRemark 强制重新标记前进行一次MinorGC，如果Eden区有大量活跃对象推荐使用。
-XX:+PrintGCDetails 输出GC的详细日志
-XX:+PrintGCTimeStamps 输出GC的时间戳
-XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息
-XX:+PrintTenuringDistribution 指定JVM在每次新生代GC时，输出幸存区中对象的年龄分布
#看懂GC日志
###一次典型的minorGC日志示例:

![](http://hexo-tuchuan.qiniudn.com/gcprocess.png)

###一次典型的CMS GC日志示例:

39.910: [GC 39.910: [ParNew: 261760K->0K(261952K), 0.2314667 secs] 262017K->26386K(1048384K), 0.2318679 secs]
新生代使用 (ParNew 并行)回收器。新生代容量为261952K，GC回收后占用从261760K降到0,耗时0.2314667秒。(译注：262017K->26386K(1048384K), 0.2318679 secs 表示整个堆占用从262017K 降至26386K,费时0.2318679)

40.146: [GC [1 CMS-initial-mark: 26386K(786432K)] 26404K(1048384K), 0.0074495 secs]
开始使用CMS回收器进行老年代回收。初始标记(CMS-initial-mark)阶段,这个阶段标记由根可以直接到达的对象，标记期间整个应用线程会暂停。
老年代容量为786432K,CMS 回收器在空间占用达到 26386K 时被触发

40.154: [CMS-concurrent-mark-start]
开始并发标记(concurrent-mark-start) 阶段，在第一个阶段被暂停的线程重新开始运行，由前阶段标记过的对象出发，所有可到达的对象都在本阶段中标记。

40.683: [CMS-concurrent-mark: 0.521/0.529 secs]
并发标记阶段结束，占用 0.521秒CPU时间, 0.529秒墙钟时间(也包含线程让出CPU给其他线程执行的时间)

40.683: [CMS-concurrent-preclean-start]
开始预清理阶段
预清理也是一个并发执行的阶段。在本阶段，会查找前一阶段执行过程中,从新生代晋升或新分配或被更新的对象。通过并发地重新扫描这些对象，预清理阶段可以减少下一个stop-the-world 重新标记阶段的工作量。

40.701: [CMS-concurrent-preclean: 0.017/0.018 secs]
预清理阶段费时 0.017秒CPU时间，0.018秒墙钟时间。

40.704: [GC40.704: [Rescan (parallel) , 0.1790103 secs]40.883: [weak refs processing, 0.0100966 secs] [1 CMS-remark: 26386K(786432K)] 52644K(1048384K), 0.1897792 secs]
Stop-the-world 阶段,从根及被其引用对象开始，重新扫描 CMS 堆中残留的更新过的对象。这里重新扫描费时0.1790103秒，处理弱引用对象费时0.0100966秒，本阶段费时0.1897792 秒。

40.894: [CMS-concurrent-sweep-start]
开始并发清理阶段，在清理阶段，应用线程还在运行。

41.020: [CMS-concurrent-sweep: 0.126/0.126 secs]
并发清理阶段费时0.126秒

41.020: [CMS-concurrent-reset-start]
开始并发重置

41.147: [CMS-concurrent-reset: 0.127/0.127 secs]
在本阶段，重新初始化CMS内部数据结构，以备下一轮 GC 使用。本阶段费时0.127秒

#一些case
###频繁MinorGC和MajorGC
某应用配置xms=6g xmx=6g xmn=4g，高峰期应用每10s发生一次Minor GC，每次GC时间在10-20ms，每40min发生一次Major GC，每次GC时间在100ms-200ms。配置成xms=10g xmx=10g xmn=7g后，高峰期应用每30s发生一次Minor GC，每次GC时间在20-30ms，每天发生一次Major GC，每次GC时间在100ms-200ms。 通常情况下，由于新生代空间较小，Eden区很快被填满，就会导致频繁Minor GC，因此可以通过增大新生代空间来降低Minor GC的频率,单次Minor GC时间由以下两部分组成：T1（扫描新生代）和 T2（复制存活对象到Survivor区),扩容后，Minor GC时增加了T1（扫描时间），但省去T2（复制对象）的时间，更重要的是对于虚拟机来说，复制对象的成本要远高于扫描成本，所以，单次Minor GC时间更多取决于GC后存活对象的数量，而非Eden区的大小。因此如果堆中短期对象很多，那么扩容新生代，单次Minor GC时间不会显著增加。
###偶发长时间FULLGC
某应用正常FULLGC时间是100-200ms，会偶发耗时大于1000ms的FULLGC，分析GC日志发现耗时突增主要发生在remark阶段， Remark耗时>1000ms时，新生代使用率都在75%以上。新生代中对象的特点是“朝生夕灭”，这样如果Remark前执行一次Minor GC，大部分对象就会被回收。对于这种情况，CMS提供CMSScavengeBeforeRemark参数，用来保证Remark前强制进行一次Minor GC。使用这个配置之后FULLGC的时间再也没有超过1000ms。
###必现长时间FULLGC
某应用每周发生一次FULLGC，正常时间都在500ms以下，突然在某个时间点之后发生的FULLGC耗时都超过了5000ms，耗时增加了一个数量级。分析发现GC真实的耗时是远大于用户耗时和系统耗时的，例如:[Times: user=2.56 sys=0.07, real=9.82 secs]。有些时候系统活动诸如内存换入换出（vmstat）、网络活动（netstat）、I/O （iostat）在 GC 过程中发生会使 GC 时间变长。特别是有 SWAP 区域（用 top、 vmstat 等命令可以看出）用于内存的换入换出，那么操作系统可能会将 JVM 中不活跃的内存页换到 SWAP 区域用以释放内存给线程使用（这也透露出内存开始不够用了）。内存换入换出是一个开销巨大的磁盘操作，比内存访问慢好几个数量级。在减小JVM大小，禁用swap后应用恢复正常。
###大量使用枚举导致频繁GC
某任务类应用配置xms=6g xmx=6g xmn=4g，解析文件生成对象进行处理，大量发生MinorGC和FULLGC，调整jvm参数后改善有限，执行jmap -histo:live [pid] 后发现大量枚举对象，应用中大量使用枚举以及valueOf方法生成枚举。使用静态map替换枚举的valueof方法后FULLGC的频率降低到原来的1/10。

#建议配置
首先确定一下我们的目标，降低GC对性能的影响，最小化STW的时间，每次STW的时间都小于服务最大响应时间的要求。更具体的目标就是最小化fullGC的频率及持续时间，最小化minorGC频率及持续时间。为了实现这个目标有以下几点限制：
1. 堆大小在合理的范围
2. 老年代大小在合理的范围
3. 幸存区大小在合理的范围

堆大小直接影响到GC的频率和时间，业务场景，应用和硬件都会对堆大小的设置产生影响，要想获得一个合适的大小，只能通过不断的测试调优。个人经验是响应时间优先的应用为操作系统和日志采集预留出足够的内存并且不大于16G的范围内堆设置的越大越好(如果业务量比较小可能造成浪费)。
老年代的大小直接影响FULLGC的频率和时间，个人经验是设置成高峰期老年代活跃数据的大小的3倍。
幸存区大小会影响晋升到老年代对象的数量，minorGC的持续时间。个人经验是幸存区的大小需要能够容纳存活的对象到达配置的MaxTenuringThreshold年龄，避免promotion failed。

参考:
[从实际案例聊聊Java应用的GC优化](https://tech.meituan.com/jvm_optimize.html)
[JVM 调优 —— GC 长时间停顿问题及解决方法](http://www.importnew.com/22886.html)
[海量连接服务端jvm参数调优杂记](https://www.jianshu.com/p/051d566e110d)
