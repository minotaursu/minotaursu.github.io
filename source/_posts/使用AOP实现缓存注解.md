title: 使用AOP实现缓存注解
date: 2016-01-27 11:47:04
tags:
- cache
- annotation
- AOP
- spring
- java 

---

> 为何重造轮子  

半年前写了一个注解驱动的缓存，最近提交到了[github](https://github.com/minotaursu/cacheAnnotation)。缓存大量的被使用在应用中的多个地方，简单的使用方式就是代码先查询缓存中是否存在数据，如果不存在或者缓存过期再查询数据库，并将查询的结果缓存一段时间，缓存key通常是入参的对象或者入参对象的某些属性，有些时候还需要按照某种条件判断是否缓存。可以看到这种功能性代码和具体的业务代码混合在一起的实现方式有很大的代码冗余，即不便于维护也不灵活。使用切面的方式可以很好的抽取功能相似代码冗余的缓存代码，将缓存代码和业务代码隔离开，这样既做到了对业务的无侵入又可以灵活更换具体缓存组件。
其实从spring3之后spring就提供了@Cacheable注解，但是用起来不爽的地方还是太多，例如缓存时间是由cache本身设置的而非在每个@Cacheable注解中指定，这个粒度有点太大了；没有缓存key的前缀设置，不同方法很容易出现key冲突。

> 怎样重造轮子

鉴于spring3提供的cache注解不太能满足需求，最后决定自己写一个。目标是构造一个简单好用而不是大而全的缓存注解，整个过程陆陆续续花了3天时间，第一天确定技术方案，构建对象和对象间的关系; 第二天写具体的实现和debug; 第三天写demo和test。
确定技术方案的时候看了spring3的cache注解实现和在阿里时使用过的2个cache注解实现。最大是不同点是创建代理类的方式和动态生成cacheKey的实现。
不同的创建代理类的方式：  
* 使用MethodInterceptor+xml配置，最经典的使用方式。缺点是同一个类的方法相互调用时不会被aop拦截，需要使用AopContext.currentProxy()获取代理类。  
* 使用@AspectJ注解，可以有效的减少xml配置，缺点和MethodInterceptor相同。  
* 基于SmartInstantiationAwareBeanPostProcessor+cglib创建代理类。 

不同的生成cacheKey的方式：  
* 使用SPEL  
* 使用OGNL  
* 使用正则表达式  

最后选择了@AspectJ+SPEL的实现方式。
虽然具体的实现方式各自不同，类的调用结构和内部功能都是基本相同的。  
* cacheManager负责cache的管理，包含cache实现的list。  
* cache是具体的缓存实现，可以是redis，ehcache，memcache。  
* keyParser负责动态生成cacheKey。  
* interceptor负责注解的拦截。  
* @Cacheable，@CacheEvict等是具体的缓存注解。  

按照上述的功能划分实现相关类后，花了一天的时间来写demo和test，全部的test跑通后就可以使用了。后面增加了一个CacheOperation转换具体的注解，统一对CacheOperation进行处理，代码简化了不少。


> 实际遇到的问题

实际使用中主要遇到了2个问题，一个是interceptor中catch了所有的Exception并打印错误日志，实际上我们会在应用中定义BizException，当发生预期内的错误时会抛出BizException，而BizException是不需要被拦截打印错误日志的。另一个是问题是并发读写问题，在cache中没有缓存的时候，ThreadA从DB获取数据，ThreadB修改了数据库的数据，ThreadB删除缓存，ThreadA然后put修改之前的数据。原本以为按照业务特点发生并发读写的概率不高，结果发现接口轮询+事务导致频繁发生不一致的情况。缓存失效策略一直是缓存使用中的难题，甚至是计算机科学中两大难题之一。处理数据库并发最常见的2个解决思路是乐观锁和串行化，但是并不适用于解决缓存和数据库的不一致，google了一下也没有找到特别好的解决方案。考虑到应用并没有超高的QPS，短时间的缓存穿透不会造成系统的崩溃，最后通过增加一个redis的缓存删除标识进行解决，这个删除标识会存活5s，在这5s中不会执行put缓存操作从而避免了缓存和数据库的不一致。


