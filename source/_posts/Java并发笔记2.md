---
title: Java并发笔记2
date: 2018-07-03 18:06:42
tags: [Java,编程语言,读书笔记,并发]
categories: [编程语言]
---
### 线程组和未处理异常
*较少使用*

- 线程组可以批量设置管理线程，如果创建的线程不指定分组则为默认线程组，默认情况下子线程和创建它的父线程属于同一个分组。
- 可以在线程或者线程组上设置`Thread.UncaughtExceptionHandler`,在线程执行过程中如果抛出一个未处理的异常，JVM在结束该线程前会自动查找是否有对应的实现了`Thread.UncaughtExceptionHandler`接口的对象，如果找到则执行该对象的`UncaughtException`方法
### Callable和Future
