---
title: maven私服搭建
date: 2018-07-31 13:16:15
tags: [maven, 项目管理, 构建]
categories: [项目管理]
---
### 1. 私服搭建的意义
1. 是为了进行仓库的管理和控制，以及减少对中心仓库的访问，减少网络请求提高效率。
2. 以及为内网用户在无法访问外网却需要使用`maven`功能
3. 发布自己的组件到私服

### 2. 所用工具`Nexus Repository Manager (NXRM)`
`NXRM`是一个仓库管理工具，能够实现上节所提到的三种特性，它不仅支持maven还支持其他多种仓库形式:
> Repository Formats:
- Maven Repositories with Apache Maven and Other Tools
- .NET Package Repositories with NuGet
- Private Registry for Docker
- Node Packaged Modules and npm Registries
- Bower Repositories
- PyPI Repositories
- Ruby, RubyGems and Gem Repositories
- Raw Repositories, Maven Sites and More
- Git LFS Repositories
- Yum Repositories

另外，`NXRM`还提供有web用户界面。

### 3. `NXRM`服务器的搭建
`NXRM`支持`*unix`,`OSX`,`windows`,`Docker`等多种平台。最新版本是`Nexus Repository Manager (NXRM) 3`，此次使用的平台及版本为`nexus-3.13.0-01-unix`。

#### 3.1 linux下安装
此次安装和配置主要参考文章:[Linux 使用 Nexus3.x 搭建 Maven 私服指南](https://blog.csdn.net/smartbetter/article/details/55116889)

在安装之前首先参考如下不同之处：
##### 3.1.1 其中所用版本略有不同
1. [下载页面](https://help.sonatype.com/repomanager3/download)
2. 选择`nexus-3.13.0-01-unix`版本，进行下载

##### 3.1.2 默认`8081`端口修改
1. 将`$NEXUS_HOME/nexus-3.13.0-01/etc/nexus-default.properties`中的`application-port=8081`修改为想要的端口，如`application-port=9001`
