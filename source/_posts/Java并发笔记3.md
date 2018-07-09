---
title: Java并发笔记3
date: 2018-07-09 14:15:15
tags: [Java,编程语言,读书笔记, Java并发编程实战]
categories: [编程语言]
---
### Java并发编程实战第二章 线程安全
编写线程安全代码的核心是管理对状态的访问，尤其*共享*和*可变*状态的访问。通过**同步**协调对对象可变状态的访问可以实现一个对象的线程安全。同步最基本的机制是`synchronized`关键词，但是同步(synchronization)这个属于还包括:`volatile`变量、显式(explicit)锁和原子(atomic)变量。

#### 最佳实践
1. 在设计时就考虑线程安全
2. 利用封装(encapsulation)，不可变性(immutablility)，清晰指定不可变量(invariants)是比较好的实践。

### 2.1 什么是线程安全
