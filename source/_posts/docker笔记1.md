---
title: docker笔记1
date: 2018-08-08 13:07:05
tags: [docker, 项目管理, 构建]
categories: [项目管理]
---
### Docker是什么
- `Docker` 是一个执行操作系统级别虚拟化的软件(`operating-system-level virtualization`),此技术也被成为容器化(`containerization`),简而言之就是一个可以运行应用的容器。
- `Docker` 是一个开源项目，诞生于 2013 年初，最初是 dotCloud 公司内部的一个业余项目。它基于 Google 公司推出的 `Go` 语言实现。 项目后来加入了 Linux 基金会，遵从了 Apache 2.0 协议，项目代码在 GitHub 上进行维护。
#### 容器(`Container`)是什么
- 容器是将需要的软件打包的一个标准单元。
- 一个`Docker`容器镜像是一个轻量级的、独立的、可执行的软件包，包中包括应用运行所需要的一切:代码、运行时、系统统计、系统类库和设置。
#### 容器的运行机制
- 容器镜像在运行时就成为一个容器，具体到`Docker`的情形就是: 当镜像在`Docker Engine`上运行时就成为了容器。
- 无论基础架构如何，容器化软件都能始终运行如一，且适用于基于Windows和Linux的应用。

### Docker能做什么
`Docker` 项目的目标是实现轻量级的操作系统虚拟化解决方案。 `Docker` 的基础是 `Linux` 容器（`LXC`）等技术。在 `LXC` 的基础上 `Docker` 进行了进一步的封装，让用户不需要去关心容器的管理，使得操作更为简便。用户操作 `Docker` 的容器就像操作一个快速轻量级的虚拟机一样简单

### Docker的优势是什么
`Docker`主要是用来代替传统虚拟化方式(虚拟机)技术的，有以下特点
- 容器是在**操作系统层面**上实现虚拟化，直接复用本地主机的操作系统，而传统方式则是在**硬件层面**实现
- 更快速的交付和部署
- 高效的部署和扩容
- 更高的资源利用率
- 更简单的管理

### Docker核心概念
#### 镜像(images)
Docker 镜像（Image）就是一个只读的模板。镜像可以用来创建 Docker 容器，一个镜像可以创建很多容器。Docker 提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用

### 参考
1. [几张图帮你理解 docker 基本原理及快速入门](https://www.cnblogs.com/SzeCheng/p/6822905.html)
2. [Docker维基百科](https://en.wikipedia.org/wiki/Docker_(software))
3. [Docker官网:What is a Container](https://www.docker.com/resources/what-container)
