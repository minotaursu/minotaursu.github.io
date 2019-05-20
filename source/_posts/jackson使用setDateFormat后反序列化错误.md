title: jackson使用setDateFormat后反序列化错误
date: 2017-12-22 12:08:57
tags:
- java
- jackson
- 语言陷阱 

---

![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/jackson.png)

最近marketing-activity系统在接入降级服务，出现启动时可以正确获取到降级配置，系统运行一段时间后修改降级策略不生效的问题，之前订单系统，用户系统和其他系统的接入都没问题出现这个问题，肯定是触发了某种特定的case。营销的同学联系我进行排查，排查日志发现是降级服务反序列化Date类型异常，接收到的数据格式是yyyy-MM-dd HH:mm:ss，并不在jackson支持的反序列化格式之内。降级服务的sdk也是使用的jackson进行序列化的，为什么会出现jackson序列化后的数据却不能使用jackson反序列化。
接着查看marketing-activity系统的降级服务sdk日志发现一个很诡异的现象，最初发送的序列化后的请求还是时间戳格式的，运行一段时间后就变成了yyyy-MM-dd HH:mm:ss格式，也就是说系统的行为在运行时被改变了。
![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/sds-log.png)

那么到底是什么可以改变系统运行时的序列化逻辑呢？可能出现的主要原因有2种。一种是字节码技术，也就是btrace，greys这些。另一种就是调用jackson本身的api改变了一些属性。显然第二种的可能性更大一些，果然在jsonUitl里发现了蛛丝马迹，toObject允许设置特定的时间格式进行反序列化，调用setDateFormat会导致后续全部的Date类型的序列化都会是yyyy-MM-dd HH:mm:ss格式，自然不能默认设置的jackson反序列化。
![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/jsonutil.png)

至此降级服务改变策略后不生效的根本原因就得到了解答，那么如果某个对象就是要使用yyyy-MM-dd HH:mm:ss 进行序列化和反序列化怎么办，建议使用@JsonFormat 单独对属性进行注释。
最后谈谈fastjson和jackson，貌似jackson的各种坑遇到过很多，而fastjson的坑很少，那么为什么jackson还是要比fastjson更流行，真的只是国外的月亮更圆，空气更甜？这个主要是json框架的设计理念偏重点不同，fastjson偏重的是简单和快速，内部实现有很多的hack和magic code。而jackson偏重的是标准和强大，格式支持json，xml，有很多的属性可以设置非常的灵活，也有很多的接口可以自定义进行扩展，导致学习成本比较高，需要详细看过jackson文档才能上手。
