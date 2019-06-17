# Logback 介绍 -- 官网中文版

`Logback`的目标是替代 `Log4j`,Logback 修复了很多 Log4j的不足。

Logback 可以被应用在多个不同的场景中。目前，logback 被划分为3个模块：

- logback-core：为其他两个模块提供基础；
- logback-classic：此模块自然的实现了 SLF4J 的 API，因此你也自由的在 log4j 或者是 java.util.logging(JUL)之间做切换；
- logback-access：这个模块用来和 Servlet 容器集成工作，比如 Tomcat 和 Jetty，它提供了 Http-access 日志打印能力。

当然你也可以在 logback-core 模块上构建自己的模块。

### Logback VS Log4j

`Logback` 和 `Log4j` 是同一个作者。

- 早在 1996 年 `Ceki` 就创立了 `Log4j`；
- 后来 `Ceki` 将 `Log4j` 捐献给了 `Apache` 基金会，称为了 `Apache` 的子项目；
- 再后来 `Apache` 宣布放弃了放弃 `Log4j` 推荐大家使用 `Log4j2`;
- 后来 `Ceki` 宣布开源 `logback` 并宣称 `logback` 是 `log4j` 的继任者；

#### log4j2 
相比 log4j 和  logback , log4j2 名气不大，它基本上把 log4j 的内核全部重写了。

### SLF4J VS Commons-Logging

这两个只是一个门面模式，是一个日志接口，也就是我们编程时直接依赖的 API；
这种机制类似 `SPI` ， 这两个框架只定义基本接口，谁来实现，怎么实现不关心，符合标准即可。

- Commons-Logging 最早叫  Jakarta Commons Logging（简称 JCL），后来捐献给 Apache 叫 Commons-Logging;
- 后来 Apache 说服 `Ceki` 使用 `JCL`；
- 再后来，`Ceki` 发现 `JCL` 在类加载上一致有Bug 不好用，自己创立了`SLF4J`;
