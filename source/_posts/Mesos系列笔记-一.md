---
title: Mesos系列笔记(一)
date: 2018-05-07 17:38:53
tags: [mesos,mesos实战,读书笔记]
categories: [大数据]
---
### mesos的起源及其所要解决的问题
2010 年， 一个旨在解决扩容问题的项目在美国加州大学伯克利分校诞生。这个软件项目就是现在所知的 `Apache Mesos`。它是一个开源的集群管理工具。它让运维和开发人员更
多地关注应用本身，而不是其下的服务器资源

### mesos的基本运行机制
- 它在某种程度上对 CPU、内存、磁盘资源进行抽象，从而允许整个数据中心如同单台大服务器般运转。无需虚拟机和操作系统， Mesos 创造了 一个单独底层的集群为应用提供所需资源。
- `Mesos`的资源隔离机制支持多用户，该功能允许多个应用运行在同一个机器上。
- `Mesos`还从一开始就支持分布式、高可用和容错。通过使用容器技术如`linux` `groups`(`cgroups`)和`Docker`，`Mesos`实现了进程间的隔离，从而允许多个应用运行在同一机器上。也就是说不必再为`Memecached`，`Jenkins CI`和`Ruby`分别去搭建集群了。
#### 如何工作
![Mesos:How to work](/images/mesos_how_to_run.png)
tip:现在`slave`已经改为`agent`
##### `Resource offers`
与其他集群管理器类似，`Mesos`集群也由一组称为`master`和`agent`的机器组成。`agent`定期以`Resource offers`的形式将可用的`cpu`,`memory`,`storage`通知到`master`。
##### 两层调度(`two-tier scheduling`)
在`Mesos`集群中，`Mesos`的分配模块(`alloction module`)和框架的调度器一起负责对资源的调度，也就是所谓的`双层调度`.
##### 资源隔离
通过`Linux cgroups`或者`Dcoker`容器，`Mesos`支持`多租户`功能，也就是允许多个进程同时在一台`agent`上运行。
