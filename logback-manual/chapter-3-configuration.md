# 章节3：Logback 配置
> 源文件链接： [Logback Configuration](https://logback.qos.ch/manual/configuration.html)

In symbols one observes an advantage in discovery which is greatest when they express the exact nature of a thing briefly and, as it were, picture it; then indeed the labor of thought is wonderfully diminished.

—GOTTFRIED WILHELM LEIBNIZ

本章我们将介绍 `logback` 的配置方法，本章会有很多配置脚本的示例。稍后的章节我们将会介绍 `logback` 的配置框架：`Joran`。

## Logback 的配置

在应用代码中插入日志输出语句需要做一些规划和努力。观察表明，程序中大约 4% 的代码是日志输出语句。因此，即使是一个中等规模的应用程序也会包含数千行的日志输出语句。基于这种数据量级考虑，我们需要一个能管理日志语句的工具。

可以使用程序代码或者是 XML、Groovy 脚本的格式来配置 `logback`。顺便说一句，`log4j` 的用户可以使用我们提供的 [`PropertiesTranslator` 应用程序](http://logback.qos.ch/translator/)将 `log4j.properties` 转换为 `logback.xml` 文件。

现在，让我们看一下 `logback` 尝试的初始化步骤：

1. `logback` 尝试在 `classpath` 中搜索名叫 `logback-test.xml` 文件；
2. 如果没有发现 `logback-test.xml` 会尝试在 `classpath` 中搜索 `logback.groovy` 文件。
3. 如果还没有找到，会尝试在 `classpath` 中搜索 `logback.xml` 文件。
4. 如果上面的文件都没找到，`logback` 会通过 `service-provider loading facility`（JDK 1.6 中引入的 `SPI` 技术）解析类路径下的 `META-INF\services\ch.qos.logback.classic.spi.Configurator` 文件，并找到实现了 `com.qos.logback.classic.spi.Configurator` 接口的类，文件中要对的类配置全限定名。
5. 如果上面的步骤都没成功，`logback` 会自动的使用 `BasicConfigurator` 来初始化它，这个配置会将日志输出到 `console`。

最后一步的意识是：做最后的努力，使用一个默认的日志输出配置（这个配置非常简单基础）来提供日志输出能力。

如果你使用 `Maven` 并且你将 `logback-test.xml` 文件放在了 `src/test/resources` 文件夹下， `Maven` 会确保它不会发布到生成包中。因此，你可以在不同地方使用不同的配置文件。在单测的时候使用 `logback-test.xml` ，在生产环境中使用 `logback.xml` 。

`FAST START-UP`(快速启动)：`Joran` 解析一个 logback 配置文件大约需要 100 毫秒。要在应用程序启动时缩短这些毫秒，可以使用 `service-provider loading facility` （上面第4点提到的）来加载您自己的自定义配置器类，扩展 `BasicConfigrator` 是一个很好的起点。


## 自动配置 logback

logback 的最简单配置方法就是让 logback 在初始化查找时失败降级为：使用默认配置。我们可以在下面的 `MyApp1` 中体验下这是如何实现的。

#### Example: Simple example of BasicConfigurator usage 
```java
package chapters.configuration;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MyApp1 {
  final static Logger logger = LoggerFactory.getLogger(MyApp1.class);

  public static void main(String[] args) {
    logger.info("Entering application.");

    Foo foo = new Foo();
    foo.doIt();
    logger.info("Exiting application.");
  }
}
```

这个类定义了一个静态 logger 变量。然后它实例化了一个 `Foo` 类的对象。 `Foo` 类如下：

#### Example: Small class doing logging 
```java


package chapters.configuration;
  
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
   
public class Foo {
  static final Logger logger = LoggerFactory.getLogger(Foo.class);
  
  public void doIt() {
    logger.debug("Did it again!");
  }
}
```

假设没有提供 `logback-test.xml` 或者 `logback.xml` 配置文件。 `logback` 会默认调用 `BasicConfigurator` 做最小化配置。这个最小的配置会将一个 `ConsoleAppender` 添加到 root logger 中。输出使用 `PatternLayoutEncoder` 来格式化，模式为： `%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n` ， 默认的 root logger 的 level 为 DEBUG。

因此，命令行的输出类似：

```
16:06:09.031 [main] INFO  chapters.configuration.MyApp1 - Entering application.
16:06:09.046 [main] DEBUG chapters.configuration.Foo - Did it again!
16:06:09.046 [main] INFO  chapters.configuration.MyApp1 - Exiting application.
```

> 除了配置 `logback` 的代码，客户端代码不要直接依赖 `logback` 代码。使用 `logback` 作为日志框架的应用，应该在编译期依赖 `SLF4J` 不要依赖 `logback` 。

`MyApp1` 应用通过调用 `org.slf4j.LoggerFactory` 和 `org.slf4j.Logger` 来链接到 `logback`  并检索到它希望使用的 logger 。注意 `Foo` 类和 `logback` 的唯一依赖是通过：`org.slf4j.LoggerFactory` 和 `org.slf4j.Logger` 类完成的。除了 `logback` 配置代码外，客户端代码不要依赖 `logback` 代码。`SLF4J` 允许在其抽象层下使用任何日志框架，它能很容易的从一个日志框架切换到另外一个日志框架。

## 使用 logback-test.xml 和 logback.xml 自动配置 logback

正如前面所提到的，logback 会尝试使用类路径中下的 `logback-test.xml` 或者 `logback.xml` 来自动配置它。下面示例的代码等同使用 `BasicConfigurator` 的默认配置：


#### Example: Basic configuration file 

```
<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>

```

将这个文件命名为 `logback.xml` (或者 `logback-test.xml`)并把它放到类路径下。启动 `MyApp1` 可以获得与前一次运行相同的结果。

## 自动打印 warning 或者 errors 时的状态信息

> 如果解析配置文件出现告警或者错误，`logback` 会自动的将它的内部状态信息打印到控制台。为了避免重复打印，如果用户显示的注册了`状态监听器`（看下面的代码示例），自动打印状态的功能会被关闭。


In the absence of warnings or errors, if you still wish to inspect logback's internal status, then you can instruct logback to print status data by invoking the print() of the StatusPrinter class. The MyApp2 application shown below is identical to MyApp1 except for the addition of two lines of code for printing internal status data.

在没有警告或错误的情况下，如果仍然希望检查 `logback` 的内部状态，可以通过调用 `StatusPrinter` 类的 `print()` 方法来 打印 `logback` 状态数据。下面所示的 `MyApp2` 应用程序与 `MyApp1` 相同，只是添加了两行代码用于打印内部状态数据。

#### Example: 打印 logback 的内部状态信息

```
public static void main(String[] args) {
  // assume SLF4J is bound to logback in the current environment
  LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
  // print logback's internal status
  StatusPrinter.print(lc);
  ...
}
```

如果一些运行良好，你会在控制台看到以下信息：

```
17:44:58,578 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Found resource [logback-test.xml]
17:44:58,671 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - debug attribute not set
17:44:58,671 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - About to instantiate appender of type [ch.qos.logback.core.ConsoleAppender]
17:44:58,687 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - Naming appender as [STDOUT]
17:44:58,812 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - Popping appender named [STDOUT] from the object stack
17:44:58,812 |-INFO in ch.qos.logback.classic.joran.action.LevelAction - root level set to DEBUG
17:44:58,812 |-INFO in ch.qos.logback.core.joran.action.AppenderRefAction - Attaching appender named [STDOUT] to Logger[root]

17:44:58.828 [main] INFO  chapters.configuration.MyApp2 - Entering application.
17:44:58.828 [main] DEBUG chapters.configuration.Foo - Did it again!
17:44:58.828 [main] INFO  chapters.configuration.MyApp2 - Exiting application.
```

在日志的最后，你可以看到和前面的例子一样的输出。你也应该注意到 `logback` 的内部消息，`Status` 对象能很方面的访问到 `logback` 的内部状态。

## 状态数据
将

Enabling output of status data usually goes a long way in the diagnosis of issues with logback. As such, it is highly recommended and should be considered as a recourse of first resort.
Instead of invoking StatusPrinter programmatically from your code, you can instruct the configuration file to dump status data, even in the absence of errors. To achieve this, you need to set the debug attribute of the configuration element, i.e. the top-most element in the configuration file, as shown below. Please note that this debug attribute relates only to the status data. It does not affect logback's configuration otherwise, in particular with respect to logger levels. (If you are asking, no, the root logger will not be set to DEBUG.)

Example: Basic configuration file using debug mode (logback-examples/src/main/resources/chapters/configuration/sample1.xml)

View as .groovy
<configuration debug="true"> 

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender"> 
    <!-- encoders are  by default assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
Setting debug="true" within the <configuration> element will output status information, assuming that:

the configuration file is found
the configuration file is well-formed XML.
If any of these two conditions is not fulfilled, Joran cannot interpret the debug attribute since the configuration file cannot be read. If the configuration file is found but is malformed, then logback will detect the error condition and automatically print its internal status on the console. However, if the configuration file cannot be found, logback will not automatically print its status data, since this is not necessarily an error condition. Programmatically invoking StatusPrinter.print() as shown in the MyApp2 application above ensures that status information is printed in every case.

FORCING STATUS OUTPUT In the absence of status messages, tracking down a rogue logback.xml configuration file can be difficult, especially in production where the application source cannot be easily modified. To help identify the location of a rogue configuration file, you can set a StatusListener via the "logback.statusListenerClass" system property (defined below) to force output of status messages. The "logback.statusListenerClass" system property can also be used to silence output automatically generated in case of errors.

By the way, setting debug="true" is strictly equivalent to installing an OnConsoleStatusListener. Status listeners are disccussed further below. The installation of OnConsoleStatusListener is shown next.

Example: Registering a status listener (logback-examples/src/main/resources/chapters/configuration/onConsoleStatusListener.xml)

View as .groovy
<configuration>
  <statusListener class="ch.qos.logback.core.status.OnConsoleStatusListener" />  

  ... the rest of the configuration file  
</configuration>
Enabling output of status data, either via the debug attribute or equivalently by installing OnConsoleStatusListener, will go a long way in helping you diagnose logback issues. As such, enabling logback status data is very highly recommended and should be considered as a recourse of first resort.

