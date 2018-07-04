---
title: Java并发笔记2
date: 2018-07-03 18:06:42
tags: [Java,编程语言,读书笔记,并发]
categories: [编程语言]
---

### 1 线程组和未处理异常
*较少使用*

- 线程组可以批量设置管理线程，如果创建的线程不指定分组则为默认线程组，默认情况下子线程和创建它的父线程属于同一个分组。
- 可以在线程或者线程组上设置`Thread.UncaughtExceptionHandler`,在线程执行过程中如果抛出一个未处理的异常，JVM在结束该线程前会自动查找是否有对应的实现了`Thread.UncaughtExceptionHandler`接口的对象，如果找到则执行该对象的`UncaughtException`方法

### 2 Callable和Future
特点如下：
1. 实现`Callable`接口后，可以通过`call()`抛出异常和返回结果
2. 实现`Future`接口的`FutureTask`可以接受`Callable`的返回值.

### 3 线程池
`ExecutorService pool = Executors.newFixedThreadPool(6);
pool.submit(new RunnableThread());//支持Runnable和Future
pool.shutdown();`

### 4 线程相关类
#### 4.1 `ThreadLocal`
1. 要点: 同步机制是为了使所有线程能够*共享*对象，而`ThreadLocal`是为了*隔离*对象，即每个线程都修改该对象的副本而不会影响到其他线程。
2. 使用: `private ThreadLocal<String> localVar= new ThreadLocal<String>()`

#### 4.2 包装线程不安全的类
`Collections`提供了以下几个接口来将线程不安全的集合转为线程安全的:
- `synchronizedCollection()` 返回线程安全的`collection`
- `synchronizedSet()` 返回线程安全的`Set`
- `synchronizedMap()` 返回线程安全的`Map`
- `synchronizedSortedSet()` 返回线程安全的`SortedSet`
- `synchronizedSortedMap()` 返回线程安全的`SortedMap`
- `synchronizedList()` 返回线程安全的`List`

#### 4.3 线程安全的集合类
`java.util.concurrent`包下的:
- `ConcurrentHashMap`
- `ConcurrentLinkedDeque`
- `ConcurrentLinkedQueue`
以上几个类都支持并发读写，在读时不会锁定。*创建迭代器之后，对其修改是不能在遍历中反应出来的，而且不会报错*,而`java.util`包下的集合类是会报`ConcurrentModificationException`的异常的
