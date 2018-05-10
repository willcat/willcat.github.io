---
title: flink系列笔记(一)入门篇
date: 2018-05-09 12:02:56
tags: [flink]
categories: [大数据]
---

### flink是什么
Flink是一个基于流计算的分布式引擎。
### flink简要历史
2010年，一项名为"Stratosphere: Information Management on the Cloud"的研究项目由柏林科技大学、柏林洪堡大学、Hasso-Plattner-Institut Potsdam合作发起。Flink从Stratosphere分布式执行引擎的分支开始，并于2014年3月成为Apache孵化器项目。在2014年12月， Flink被纳入Apache顶级项目
### flink的特性
1. 一个流式优先的运行时，它同时支持批处理和数据流式程序
2. 同时支持Java和Scala的优雅、流畅的API
3. 一个同时支持高吞吐和低延迟的运行时
4. 支持基于`Dataflow`模型的事件时间(`event time`)处理和乱序处理
5. 跨越不同时间语义(`event time`,`processing time`)的灵活的时间窗口(`time`,`count`,`session`,`custom triggers`)
6. 确保`Exactly Once`的容错性
7. 流式程序中的自然`back-pressure`
8. 各种类库: 图处理 (batch), Machine Learning (batch), and Complex Event Processing (streaming)
9. 在DataSet(batch)API中内置对迭代程序(BSP)的支持
10. 定制内存管理功能，可在内存和外核数据处理算法之间进行高效稳健的切换
11. Apache Hadoop MapReduce和Apache Storm的兼容层
12. 与YARN，HDFS，HBase以及Apache Hadoop生态系统的其他组件集成

### 预备概念

#### 两种数据集类型
- 无界数据。持续追加的无限数据集。
- 有界数据。确定的、不会改变的数据

#### 两种执行模型
- Streaming
- Batch

### 流处理技术的发展
连续数据的生产和批处理数据的消耗之间的脱节，虽然使系统构建者的工作变得容易，但是对应用开发者和DevOps团队等需要使用和管理这些系统的使用者来说，这增加了他们管理的复杂度。
为了解决这个脱节，一个开路先是`Apache Strom`项目，该项目在被纳入ASF之前，最初由Nathan Marz和一个叫BackType(后来被Twitter收购)的公司的一个团队发起。Storm带来了低延迟的流处理特性，但是它的实时处理带有折衷即：很难达到高吞吐，而且Storm没有提供通常需要的容错性(correctness)水平。换言之，Storm 缺少在维护正确状态时的`exactly-once`保证，而且Storm能提供的正确性保证也有很大的开销。

#### Lambda架构概览:优缺点
分布式架构如HDFS和基于批处理的MapReduce实现了可用性的扩展，但是这种方式很难解决低延迟的insight。Storm解决了低延迟的问题，但是还不是一个完整的方案，Storm不能通过exactly-once处理保证状态一致性，而且不太能解决时间处理(event-time processing)。
一种融合了这些方案的数据分析的混合观点提供了解决这些挑战的一种途径。这种混合就叫:`Lambda架构`,通过批处理MapReduce job提供延迟的但是准确的的结果，通过Storm提供一种及时和初步的新结果的视图.\
但是 Lambda也有它的问题所在，但一个时间窗口内由于可见的错误导致不准确数据产生时，Lambda架构需要为相同的业务逻辑编码两次，一是批处理系统，一是流式处理系统。
Spark Stream使用的是“微批”方法，不能达到完全的实时，不过可以使延迟降到几秒甚至亚秒。但是，作为低延迟/实时的代价，无法将窗口适配自然发生的会话，在表达性方面也遇到一些挑战。
