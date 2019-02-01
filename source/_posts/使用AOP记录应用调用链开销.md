title: 使用AOP记录应用调用链开销
date: 2017-04-12 12:47:04
tags:
- 性能 
- AOP
- spring
- java 

---

最近系统出现了一次线上的性能问题，本来以为目前的QPS应该是不会出现任何问题的，结果微服务还是比较容易因为某个点的问题导致雪崩的。。。出了性能问题就要做分析，正统的思路是要不断进行压测用JProfiler进行分析。后来自己简单搞了一下使用AOP抓取调用树和开销，看起来效果还不错，加上动态开关可以偶尔在线上用一下。代码提交到了[github](https://github.com/minotaursu/profilerAop)。本身的实现类似树的深度优先遍历，一个节点有多个子节点，在进入方法之前enter，在退出方法后release，都被release了就可以打印调用树日志了。而webx的profiler本身就提供了这种实现，大大的减少了开发时间。虽然之前在使用webx的时候总是觉得不爽，没有springmvc来的简洁，layout,action,screen也不适合移动时代的开发，现在都是rest服务或者使用api gateway配置api了，但不得不说webx的很多思想还是值得深入学习的，很多工具也很适合开源使用。一个框架能够稳定运行在各种业务场景，大范围推广使用本身就是了件不起的事情，这里给webx点个赞。
最后来看一下profiler的demo效果。
![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/profiler.png)

