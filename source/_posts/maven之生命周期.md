---
title: maven之生命周期
date: 2018-07-27 09:47:07
tags: [maven, 项目管理, 构建]
categories: [项目管理]
---
### maven生命周期
Maven将项目的清理、编译、测试、部署等构建过程的各个阶段进行了抽象，形成了三个相互独立的生命周期:
#### `clean`生命周期
主要负责项目的清理工作。包括以下几个顺序阶段:
1. pre-clean    ：执行清理前的工作；
2. clean    ：清理上一次构建生成的所有文件；
3. post-clean    ：执行清理后的工作

#### `default`生命周期
负责项目的部署，`default`生命周期是最核心的，它包含了构建项目时真正需要执行的所有步骤。它包含以下各个阶段:
1. validate
2. initialize
3. generate-sources
4. process-sources
5. generate-resources
6. process-resources    ：复制和处理资源文件到target目录，准备打包；
7. compile    ：编译项目的源代码；
8. process-classes
9. generate-test-sources
10. process-test-sources
11. generate-test-resources
12. process-test-resources
13. test-compile    ：编译测试源代码；
14. process-test-classes
15. test    ：运行测试代码；
16. prepare-package
17. package    ：打包成jar或者war或者其他格式的分发包；
18. pre-integration-test
19. integration-test
20. post-integration-test
21. verify
22. install    ：将打好的包安装到本地仓库，供其他项目使用；
23. deploy    ：将打好的包安装到远程仓库，供其他项目使用；

#### `site`生命周期
负责创建项目的文档站点等工作。
1. pre-site
2. site ：生成项目的站点文档；
3. post-site
4. site-deploy ：发布生成的站点文档

### 执行方式解释
每个生命周期都是按顺序执行，且后面的phase依赖于前面的phase。执行某个phase时，其前面的phase会依顺序执行，即`mvn cmd`指定的某个`cmd`阶段为从该生命周期顺序执行构建过程的最后一个阶段，但不会触发另外两套生命周期中的任何phase

### 测试
#### 约定优于配置：
在默认情况下，“maven-surefire-plugin”插件将自动执行项目“src/test/java”路径下的测试类，但测试类需要遵从以下命名模式，Maven才能自动执行它们：　　
- `Test*.java`：以Test开头的Java类；
- `*Test.java` ：以Test结尾的Java类;
- `*TestCase.java`：以TestCase结尾的Java类；

### 参考文章
- [Maven测试篇](https://www.cnblogs.com/liubaozhe/p/6929432.html)
- [Maven入门指南⑦：Maven的生命周期和插件](https://www.cnblogs.com/luotaoyeah/p/3819001.html)
