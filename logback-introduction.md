# Logback 介绍

原文链接： [logback-introduction](https://logback.qos.ch/index.html)

> 本节翻译加入了一些个人排版和说明；

`logback` 是 `Log4j` 的继任者，它修复了很多 `Log4j` 的不足。

`logback` 的体系结构设计的足够通用，因此它可以被应用在多个不同的场景中。目前，`logback` 被划分为3个模块：

- `logback-core`：为其他两个模块提供基础；
- `logback-classic`：此模块自然的实现了 `SLF4J` 的 API，因此你可以自由的在 `log4j` 或者是 `java.util.logging`(`JUL`)之间做切换；
- `logback-access`：这个模块用来和 `Servlet` 容器集成工作，比如 `Tomcat` 和 `Jetty`，它提供了 Http-access 日志打印能力。

当然你也可以在 `logback-core` 模块上构建自己的模块。

## 以下为非官网内容

### 日志门面 VS 日志框架
日志框架实现了日志的输出、打印、格式化等工作，而为了不让大家被日志框架绑在一条船上。业界大佬推出了日志门面。日志门面是用规定了一个日志 API 和接口，日志框架实现接口。使用者依赖日志门面。

- `slf4j` 和 `commons-logging` 是日志门面；
- `logback` 和 `log4j` 是日志框架；

### SLF4J VS Commons-Logging

- `Commons-Logging` 最早叫  `Jakarta Commons Logging`（简称 `JCL`），后来捐献给 Apache 改叫 `Commons-Logging`;
- 之后 Apache 说服 `Ceki` 使用 `JCL`；
- 再后来，`Ceki` 发现 `JCL` 在类加载上有一些 Bug 不好用，自己创立了`SLF4J` (simple log for java);
- 然后就出现了现在 `JCL` 和 `SLF4J` 两分天下的局面；
- `jcl-over-slf4j.jar` 的功能就是把 `jcl` 的日志桥接到 `slf4j` 上；

### Logback VS Log4j

`Logback` 和 `Log4j` 是同一个作者。

- 早在 1996 年 `Ceki` 就创立了 `Log4j`；
- 后来 `Ceki` 将 `Log4j` 捐献给了 `Apache` 基金会，成为了 `Apache` 的子项目；
- 再后来 `Apache` 宣布放弃了放弃 `Log4j` 推荐大家使用 `Log4j2`;
- 然后 `Ceki` 宣布创建了 `logback` 并宣称 `logback` 是 `log4j` 的继任者；

> 相比 `log4j` 和 `logback` , `log4j2` 名气不大，它源自 `log4j` 但是，它基本上把 `log4j` 的内核全部重写了。


