---
title: Effective Java读书笔记1
date: 2018-07-05 11:18:13
tags: [Java,编程语言,读书笔记,Effective Java]
categories: [编程语言]
---

### Item 7: 消除过期对象引用
产生过期(obsolete)对象的主要原因:
1. 类自己管理内存
2. 缓存处理不当
3. 监听和其他回调

#### 类自己管理内存
##### 场景
例如实现一个`Stack`，使用数组维护栈内内容，当栈增加元素时是没问题的，但是当栈执行`pop()`时，如果只返回当前元素，而不将数组当前元素指向`null`，则此对象因一直被引用而不能被回收。
##### 解决方案
1. 手动将不用的对象引用指向`null`。*注意:这样做应该是例外而非惯例*
2. 让包含对象引用的变量尽快脱离其作用域才是更好的实践，如将变量定义在最小的作用域里。

#### 缓存处理不当
##### 场景
当将对象放到缓存中，随着时间推移，缓存中的对象已经无效后未被正确清除就会造成内存泄漏。
##### 解决方案
1. 使用`WeakHashMap`存储缓存，当`key`不再被引用的时候，`GC`会自动把这个`entry`清理掉
2. 对于那些使用时间不确定的缓存条目，可以是用`ScheduledThreadPoolExecutor`在后台定时清理
3. 也可以当做添加新缓存条目时的副作用，比如通过重写`LinkedHashMap.removeEldestEntry()`方法，在添加新元素时该方法如果返回true清除旧的元素
4. 更复杂的需求则需要是用`java.lang.ref`了

#### 监听和其他回调
##### 场景
当你的API有回调函数被Client注册之后，Client如果没有显式解除注册就会造成内存泄漏
##### 方案
1. 将这些回调用`WeakHashMap`的key保存

#### 实践建议
1. 提前预防，编码时就认真审核这些问题。
2. 其次是审核代码和使用`heap profiler`来分析
