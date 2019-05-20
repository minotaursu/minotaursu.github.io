title: java保留小数点后2位格式化问题
date: 2019-04-29 14:07:43
tags:
- java
- 软件开发
- 语言陷阱
- jdk8

---

最近在做应用的jdk7升级8和核心类库升级的工作。发现一个把Double格式化成String的函数并不是每次都按照4舍5入进行格式化，例如1.325的运行结果是1.32，而12.345的运行结果则是12.35。这个问题是因为在二进制中并没有一个值能准确的表示这个十进制的小数值，只是非常接近这个十进制数值。

![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/java-bug.png)

如果希望所见即所得的获取格式化结果，可以使用BigDecimal.toPlainString或者String.format代替DecimalFormat 进行浮点数格式化。
相同的情况也存在于new BigDecimal和BigDecimal.valueof中，使用new BigDecimal(double val)会得到更精准的值，但是BigDecimal.valueof才是所见即所得。
