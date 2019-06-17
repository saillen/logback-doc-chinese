# 章节1： logback 介绍

> 士气的影响力是非常惊人的。当一个系统被运行起来的时候，即使它是一个非常简单的系统，大家也会热情高涨。当一个图形软件首次在屏幕上绘制出一个图像，即使这个图像只是一个简单的矩形，人们也会加倍努力的工作。在项目的每个阶段都设立一个可被运行起来的系统（感觉应该理解为里程碑）。这样你会发现团队能在几个月的时间完成超过预期的任务。
> 
> — 《人月神话》FREDERICK P. BROOKS, JR.


## 什么是 logback？

`logback` 是 `log4j` 的继任者。`log4j` 的发明者创造了 `logback`。作者利用十多年的工业级日志系统设计经验创造了 `logback`。 `logback` 比目前现存的所有日志系统要`更快`、`更轻量`。`logback` 提供了很多其他日志系统没有的非常有用的特性。

## 迈出第一步
### 必须的jar

`logback-classic` 模块依赖 `slf4j-api.jar` 和 `logback-core.jar`.

`logback-*.jar` 文件是 `logback` 发行版的一部分，然而 `slf4j-api-1.8.0-beta1.jar` 是 `SLF4J` 的一部分，他们是两个项目。

ok，现在让我们来体验下 `logback`。

> 为了确保能运行本章的示例，你需要确保你已经正确下载并配置了（就是把logback的jar加入到classpath）相关jar包。如果你还没做，可以去安装页面了解更多内容。

##### Example 1.1: Basic template for logging

```java
package chapters.introduction;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld1 {

  public static void main(String[] args) {

    Logger logger = LoggerFactory.getLogger("chapters.introduction.HelloWorld1");
    logger.debug("Hello world.");

  }
}

```

类 `HelloWord1` 定义在 `chapters.introduction` 包下。它先引入了 `org.slf4j` 包下的两个类：`Logger` 和 `LoggerFactory` 类，这两个类属于 `SL4FJ` 的 API。

`main()` 方法的第一行， 通过调用 `LoggerFactory` 的静态方法 `getLogger()` 我们为 logger 变量分配了一个 `Logger` 实例。 这个 logger 的名字叫做："chapters.introduction.HelloWorld1"。接下来，我们将"Hello World" 作为参数传递给了 logger 的 `debug` 方法。因此我们说：**main方法含有一个带有"Hello World"消息的、DEBUG 级别的日志语句**。

注意：上面的例子并没用使用到 `logback` 的类。就日志记录而言，绝大部分的情况，你只需要引入 `SLF4J` 的类即可。一般情况下（当然不是所有情况），你的类只需要使用 `SLF4J` 的 API 即可，不需要感知 `logback` 的存在。

你可以在命令行中运行下第一个例子：

```shell
java chapters.introduction.HelloWorld1
```

运行完成后，你会在控制条看到下面的输出消息。`logback` 的默认配置策略就是：**如果找不到默认的配置文件，logback会向 root logger 中添加 ConsoleAppender **

```
20:49:07.962 [main] DEBUG chapters.introduction.HelloWorld1 - Hello world.
```

Logback 可以通过它内建的`status system`（状态系统）报告它的内部状态信息。可以通过名为`StatusManager`的组件访问`logback`生命周期中发生的重要事件。现在，让我们通过调用`StatusPrinter`类的静态方法`print()`来指示logback打印其内部状态。

##### Example: Printing Logger Status

```java
package chapters.introduction;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import ch.qos.logback.classic.LoggerContext;
import ch.qos.logback.core.util.StatusPrinter;

public class HelloWorld2 {

  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger("chapters.introduction.HelloWorld2");
    logger.debug("Hello world.");

    // print internal state
    LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
    StatusPrinter.print(lc);
  }
}
```

运行 HelloWorld2 应用程序将产生以下输出：

```
12:49:22.203 [main] DEBUG chapters.introduction.HelloWorld2 - Hello world.
12:49:22,076 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.groovy]
12:49:22,078 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback-test.xml]
12:49:22,093 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.xml]
12:49:22,093 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Setting up default configuration.
```

Logback 解释说它没有找到 logback-test.xml 和 logback.xml 配置文件（稍后会讨论），因此它使用了一个默认的策略类初配置自己，这个默认策略是基于：`ConsoleAppender`的。 `Appender` 是一个可以看做是一个 output 的目的地（an output destination 意思就是日志需要输出、写入的地方）。Apppenders 可以被用在很多地方，比如写入到控制台、文件、Syslog、TCP 连接、JMS等等。用户也可以根据自己的目的创建属于自己的特殊的 Appenders。

注意：如果发生了错误，logback 会自动的将它的内部状态输出到控制台。

前面的例子比较简单。事实上，在大型应用中日志的记录方式不会有很大的不同。起码日志记录的模式不会有变化。不同的可能仅仅是配置，当日你可能想根据自己需要配置或者定制 logback。我们会在后面的章节介绍 logback 的配置。

注意：我们在上面的例子中已经介绍了如何通过 StatusPrinter.print() 方法打印 logback 的内部状态。 Logback 的内部状态对于诊断 logback 相关问题非常有用。

下面是在你的应用中使用 logback 必须做的三件事情：

- 配置 logback 的环境信息。你有好几种或多或少的配置方案，稍后我们会再讨论这点。
- 在每个你希望记录日志的地方，调用 `org.slf4j.LoggerFactory` 类的 `getLogger()` 获取日志记录对象，调用参数是当前的类名称或者是类对象本身。
- 调用 logger 实例的日志打印方法，根据你的需要选择调用： debug()/infor()/warn()/error() 方法。

### 构建 logback

logback 使用 Maven 作为构建工具。

如果你已经安装了 Maven ，你可以使用 mvn install 命令来构建整个logback项目的所有模块。 maven 会自动下载 logback 依赖的所有资源。

logback发行版包含完整的 logback 源码，你可以修改里面的内容并构建属于你自己的版本。在遵循 LGPL 或者是 EPL 许可的情况下，你也可以重新发行他们。

如果想在 IDE 下构建 logback ，可以参考：安装页面。