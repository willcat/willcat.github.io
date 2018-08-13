---
title: Junit最佳实践(译)
date: 2018-08-13 18:38:01
tags: [编程语言,测试,Junit,单元测试]
categories: [测试]
---
### 原文
[JUnit Best Practices](http://www.kyleblaney.com/junit-best-practices/)
### 本文针对单元测试的最佳实践目标:
1. 非常快- 你会写很多的单元测试，而且他们会被频繁的运行，所以他们需要运行的很快。我说的是秒级而非毫秒级别的快。
2. 非常可靠- 你希望通过测试找到`production`代码的问题。当且仅当`production`代码被破坏时，测试才失败。有些人认为测试间歇性失败比不测试更问题更严重。
