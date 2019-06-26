# 配置语法

Configuration file syntax

As you have seen thus far in the manual with plenty of examples still to follow, logback allows you to redefine logging behavior without needing to recompile your code. Indeed, you can easily configure logback so as to disable logging for certain parts of your application, or direct output to a UNIX Syslog daemon, to a database, to a log visualizer, or forward logging events to a remote logback server, which would log according to local server policy, for example by forwarding the log event to a second logback server.

The remainder of this section presents the syntax of configuration files.

As will be demonstrated over and over, the syntax of logback configuration files is extremely flexible. As such, it is not possible to specify the allowed syntax with a DTD file or an XML schema. Nevertheless, the very basic structure of the configuration file can be described as, <configuration> element, containing zero or more <appender> elements, followed by zero or more <logger> elements, followed by at most one <root> element. The following diagram illustrates this basic structure.

basic SyntaxIf you are unsure which case to use for a given tag name, just follow the camelCase convention which is almost always the correct convention.
Case sensitivity of tag names

Since logback version 0.9.17, tag names pertaining to explicit rules are case insensitive. For example, <logger>, <Logger> and <LOGGER> are valid configuration elements and will be interpreted in the same way. Note that XML well-formedness rules still apply, if you open a tag as <xyz> you must close it as </xyz>, </XyZ> will not work. As for implicit rules, tag names are case sensitive except for the first letter. Thus, <xyz> and <Xyz> are equivalent but not <xYz>. Implicit rules usually follow the camelCase convention, common in the Java world. Since it is not easy to tell when a tag is associated with an explicit action and when it is associated with an implicit action, it is not trivial to say whether an XML tag is case-sensitive or insensitive with respect to the first letter. If you are unsure which case to use for a given tag name, just follow the camelCase convention which is almost always the correct convention.

Configuring loggers, or the <logger> element

At this point you should have at least some understanding of level inheritance and the basic selection rule. Otherwise, and unless you are an Egyptologist, logback configuration will be no more meaningful to you than are hieroglyphics.

A logger is configured using the <logger> element. A <logger> element takes exactly one mandatory name attribute, an optional level attribute, and an optional additivity attribute, admitting the values true or false. The value of the level attribute admitting one of the case-insensitive string values TRACE, DEBUG, INFO, WARN, ERROR, ALL or OFF. The special case-insensitive value INHERITED, or its synonym NULL, will force the level of the logger to be inherited from higher up in the hierarchy. This comes in handy if you set the level of a logger and later decide that it should inherit its level.

Note that unlike log4j, logback-classic does not close nor remove any previously referenced appenders when configuring a given logger.
The <logger> element may contain zero or more <appender-ref> elements; each appender thus referenced is added to the named logger. Note that unlike log4j, logback-classic does not close nor remove any previously referenced appenders when configuring a given logger.

Configuring the root logger, or the <root> element

The <root> element configures the root logger. It supports a single attribute, namely the level attribute. It does not allow any other attributes because the additivity flag does not apply to the root logger. Moreover, since the root logger is already named as "ROOT", it does not allow a name attribute either. The value of the level attribute can be one of the case-insensitive strings TRACE, DEBUG, INFO, WARN, ERROR, ALL or OFF. Note that the level of the root logger cannot be set to INHERITED or NULL.

Note that unlike log4j, logback-classic does not close nor remove any previously referenced appenders when configuring the root logger.
Similarly to the <logger> element, the <root> element may contain zero or more <appender-ref> elements; each appender thus referenced is added to the root logger. Note that unlike log4j, logback-classic does not close nor remove any previously referenced appenders when configuring the root logger.

Example

Setting the level of a logger or root logger is as simple as declaring it and setting its level, as the next example illustrates. Suppose we are no longer interested in seeing any DEBUG messages from any component belonging to the "chapters.configuration" package. The following configuration file shows how to achieve that.

Example: Setting the level of a logger (logback-examples/src/main/resources/chapters/configuration/sample2.xml)

View as .groovy
<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <logger name="chapters.configuration" level="INFO"/>

  <!-- Strictly speaking, the level attribute is not necessary since -->
  <!-- the level of the root level is set to DEBUG by default.       -->
  <root level="DEBUG">          
    <appender-ref ref="STDOUT" />
  </root>  
  
</configuration>
When the above configuration file is given as argument to the MyApp3 application, it will yield the following output:

17:34:07.578 [main] INFO  chapters.configuration.MyApp3 - Entering application.
17:34:07.578 [main] INFO  chapters.configuration.MyApp3 - Exiting application.
Note that the message of level DEBUG generated by the "chapters.configuration.Foo" logger has been suppressed. See also the Foo class.

You can configure the levels of as many loggers as you wish. In the next configuration file, we set the level of the chapters.configuration logger to INFO but at the same time set the level of the chapters.configuration.Foo logger to DEBUG.

Example: Setting the level of multiple loggers (logback-examples/src/main/resources/chapters/configuration/sample3.xml)

View as .groovy
<configuration>

  <appender name="STDOUT"
    class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>
        %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
     </pattern>
    </encoder>
  </appender>

  <logger name="chapters.configuration" level="INFO" />
  <logger name="chapters.configuration.Foo" level="DEBUG" />

  <root level="DEBUG">
    <appender-ref ref="STDOUT" />
  </root>

</configuration>
Running MyApp3 with this configuration file will result in the following output on the console:

17:39:27.593 [main] INFO  chapters.configuration.MyApp3 - Entering application.
17:39:27.593 [main] DEBUG chapters.configuration.Foo - Did it again!
17:39:27.593 [main] INFO  chapters.configuration.MyApp3 - Exiting application.

The table below list the loggers and their levels, after JoranConfigurator has configured logback with the sample3.xml configuration file.

Logger name	Assigned Level	Effective Level
root	DEBUG	DEBUG
chapters.configuration	INFO	INFO
chapters.configuration.MyApp3	null	INFO
chapters.configuration.Foo	DEBUG	DEBUG
It follows that the two logging statements of level INFO in the MyApp3 class as well as the DEBUG messages in Foo.doIt() are all enabled. Note that the level of the root logger is always set to a non-null value, DEBUG by default.

Let us note that the basic-selection rule depends on the effective level of the logger being invoked, not the level of the logger where appenders are attached. Logback will first determine whether a logging statement is enabled or not, and if enabled, it will invoke the appenders found in the logger hierarchy, regardless of their level. The configuration file sample4.xml is a case in point:

Example: Logger level sample (logback-examples/src/main/resources/chapters/configuration/sample4.xml)