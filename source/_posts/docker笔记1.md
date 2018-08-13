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
#### 仓库(repository)
仓库（Repository）是集中存放镜像文件的场所。有时候会把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。

仓库分为公开仓库（Public）和私有仓库（Private）两种形式。最大的公开仓库是 Docker Hub，存放了数量庞大的镜像供用户下载。国内的公开仓库包括 时速云 、网易云 等，可以提供大陆用户更稳定快速的访问。当然，用户也可以在本地网络内创建一个私有仓库。

当用户创建了自己的镜像之后就可以使用 push 命令将它上传到公有或者私有仓库，这样下次在另外一台机器上使用这个镜像时候，只需要从仓库上 pull 下来就可以了。

Docker 仓库的概念跟 Git 类似，注册服务器可以理解为 GitHub 这样的托管服务。

### 参考
1. [几张图帮你理解 docker 基本原理及快速入门](https://www.cnblogs.com/SzeCheng/p/6822905.html)
2. [Docker维基百科](https://en.wikipedia.org/wiki/Docker_(software))
3. [Docker官网:What is a Container](https://www.docker.com/resources/what-container)
4. [Docker —几个概念的理解](http://www.cnblogs.com/wish123/p/5573098.html)
