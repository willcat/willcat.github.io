---
title: maven非标准结构处理
date: 2018-07-30 14:11:47
tags: [maven, 项目管理, 构建]
categories: [项目管理]
---
### maven build配置
#### 1. 非标准maven结构指定test目录
```java
<build>
    //...
    <directory>target</directory>  // 工作目录  
    <sourceDirectory>src</sourceDirectory> // 源码目录  
    <scriptSourceDirectory>js/scripts</scriptSourceDirectory> //脚本目录  
    <testSourceDirectory>test</testSourceDirectory> // 测试目录  
    <outputDirectory>target/classes</outputDirectory> // 源码编译目录，被工作目录包含  
    <testOutputDirectory>target/test/classes</testOutputDirectory>// 测试源码编译目录，被工作目录包含  
</build>

```
#### 2. 生成类的单元测试类到指定路径
[idea生成类的单元测试类到指定路径](https://www.cnblogs.com/xinziyublog/p/5694420.html)

#### 3. 怎么打包其他的非标准目录。如某个项目根目录下的my-resource资源目录
```java
<build>
    ...
    <resources>
        <resource>
            <targetPath>my-resource</targetPath>
            <directory>my-resource</directory>
            <filtering>true</filtering>//此选项是用来替换变量的，参考代码后的文档
        </resource>
    </resources>
    ...
</build>
```
[maven resource filtering](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html)
