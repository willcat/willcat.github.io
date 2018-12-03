---
title: Flink log4j
date: 2018-11-26 11:16:29
tags: [flink,log4j]
categories: [大数据,java]
---
# log4j简介
[Log4j.properties配置详解](https://www.jianshu.com/p/ccafda45bcea)
# Flink 日志
- 命令行模式配置文件`FLINK_HOME/conf/log4j-cli.properties`

# flink logging配置
## Local Mode
在本地模式中，比如在IDE中运行你的应用，你可以像往常一样配置log4j，比如在classpath中生成一个可用的`log4j.properties`。一个简单的方式就是在maven项目`src/main/resources`目录下创建`log4j.properties`。如下例:
```xml
log4j.rootLogger=INFO, console

# patterns:
#  d = date
#  c = class
#  F = file
#  p = priority (INFO, WARN, etc)
#  x = NDC (nested diagnostic context) associated with the thread that generated the logging event
#  m = message


# Log all infos in the console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{dd/MM/yyyy HH:mm:ss.SSS} %5p [%-10c] %m%n

# Log all infos in flink-app.log
log4j.appender.file=org.apache.log4j.FileAppender
log4j.appender.file.file=flink-app.log
log4j.appender.file.append=false
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{dd/MM/yyyy HH:mm:ss.SSS} %5p [%-10c] %m%n

# suppress info messages from flink
log4j.logger.org.apache.flink=WARN
```
## standalone Mode
在独立模式下，实际使用的配置文件不是`jar`包里的那个。这是因为Flink有自己的配置文件，这些文件优先于您自己的配置文件。

**默认文件**: Flink附带以下默认属性文件
- `log4j-cli.properties`: 由Flink命令行客户端使用（例如`flink run`）（不是在集群上执行的代码）
- `log4j-yarn-session.properties`:  启动YARN会话时由Flink命令行客户端使用（`yarn-session.sh`）
- `log4j.properties`: `JobManager` / `Taskmanager`日志（独立和YARN）

请注意，`${log.file}`默认为`flink/log`。可以通过设置`env.log.dir`在`flink-conf.yaml`中覆盖它。`env.log.dir`定义保存`Flink`日志的目录。它必须是一条绝对路径。

**Log location**: 日志是本地的，即它们是在运行`JobManager/Taskmanager(s)`的机器中生成的。

**Yarn**: 在Yarn上运行Flink时，您必须依赖`Hadoop YARN`的日志记录功能。最有用的功能是YARN日志聚合。要启用它，请在`yarn-site.xml`文件中将`yarn.log-aggregation-enable`属性设置为`true`。启用后，您可以使用检索（失败的）YARN会话的所有日志文件
`yarn logs -applicationId <application ID>`
不幸的是，日志仅在会话停止运行后可用，例如在失败后。
