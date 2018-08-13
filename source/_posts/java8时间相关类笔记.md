---
title: java8时间相关类笔记
date: 2018-08-13 13:56:13
tags: [Java,编程语言]
categories: [编程语言]
---
### 所在包
 `Java8`引入了一套全新的时间日期API,在`java.time`包下。

 `time`包中的是类是*不可变*且*线程安全*的

### 主要类
- `Instant` ——它代表的是时间戳
- `LocalDate`——不包含具体时间的日期，比如2014-01-14。它可以用来存储生日，周年纪念日，入职日期等。
- `LocalTime`——它代表的是不含日期的时间
- `LocalDateTime`——它包含了日期及时间，不过还是没有偏移信息或者说时区。
- `ZonedDateTime`——这是一个包含时区的完整的日期时间，偏移量是以UTC/格林威治时间为基准的。

### 参考
[java8新的时间日期库及使用示例](https://www.cnblogs.com/comeboo/p/5378922.html)
