> log4j2是Java的日志工具log4j的升级版，它继承了log4j的优点的同时对API进行了一些改变

## 结构
**log4j2的主要类图：**

![log4j2的主要类图](https://logging.apache.org/log4j/2.x/images/Log4jClasses.jpg)

### logger的层次

和普通的System.out.println相比，日志工具的首要优点就在于它能够禁用某些日志语句的同时允许其他日志语句不受阻碍的输出。

这在log4j1.x中是通过各个logger之间的关系维持的，而在log4j2.x中则是通过loggerConfig之间的关系进行维持的。

在log4j2.x中，logger和loggerConfig是命名实体，它们的命名是大小写敏感并且遵循类似于Java的包名的层次规则：

**即名为X.Y的logger是名为X的logger的子级。**

因此，我们可以使用以下两种方法获取顶层logger：
```java
Logger logger = LogManager.getLogger(LogManager.ROOT_LOGGER_NAME);
Logger logger = LogManager.getRootLogger();
```

### LoggerContext
LoggerContext充当日志记录系统的锚点。但是，根据具体情况，在应用程序中有多个活动LoggerContext是可能的。

### Configuration
每个LoggerContext都有一个被激活的Configuration。该Configuration包含所有的Appender、上下文范围的 Filter、LoggerConfig以及对StrSubstitutor的引用。在重新配置期间，将存在两个配置对象。一旦所有记录器被重定向到新配置，旧配置将停止并丢弃。

### Logger
Logger是通过调用LogManager.getLogger被创建的。Logger本身并不执行任何直接操作。

需要注意的是，如果创建时使用的是相同的字符串为Logger命名，那么获得的将是相同的Logger。
```java
Logger x = LogManager.getLogger("wombat");
Logger y = LogManager.getLogger("wombat");
//输出true
System.out.println(x.equals(y));
```

### LoggerConfig
LoggerConfig对象在Logger使用配置进行声明时被创建，它包含了一系列的Filter，并且包含对应用于处理事件的一组Appender的引用。

### 日志等级
1. **TRACE**
2. **DEBUG**
3. **INFO**
4. **WARN**
5. **ERROR**
6. **FATAL**

当然log4j2也支持用户自定义日志等级。

## 使用示例

### 配置方式
log4j主要有4中方式进行配置，在编程中，可以使用其中的任何一种进行配置：
1. 通过用XML、JSON、YAML或属性格式编写的配置文件。
2. 以编程方式，通过创建ConfigurationFactory和配置实现。
3. 以编程方式，通过调用配置接口中公开的API向默认配置添加组件。
4. 以编程方式调用内部Logger类上的方法。

### 自动配置
log4j2在启动时，它将会扫描资源文件用以配置自己，并按照内自己的加权顺序从高到低查找不同的配置文件，在这里我们使用XML文件进行配置作为演示。

### 演示
首先在资源文件夹中创建文件log4j2.xml，并写入配置文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Root level="info">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```
以下是两个测试类：
```java
public class TestLog {
    private static final Logger logger = LogManager.getLogger(TestLog.class);

    public static void main(String[] args){
        logger.info("start TestLog!");
        logger.debug("call Another.test()!");
        AnotherLog.test();
        logger.debug("return TestLog!");
        logger.info("exit TestLog!");
    }
}
```

```java
public class AnotherLog {
    private static Logger log = LogManager.getLogger(AnotherLog.class.getName());

    public static void test(){
        log.info("enter AnotherLog!");
        log.trace("you call test method!");
        log.info("exit AnotherLog!");
    }
}
```
运行结果如下：
```
22:18:20.568 [main] INFO  com.TestLog - start TestLog!
22:18:20.575 [main] INFO  com.AnotherLog - enter AnotherLog!
22:18:20.575 [main] INFO  com.AnotherLog - exit AnotherLog!
22:18:20.575 [main] INFO  com.TestLog - exit TestLog!
```
但是如果我们希望看到TestLog类的trace级别日志，但是不希望看到其他所有的trace日志该怎么办？这时候明显修改根Logger是不行的，所以我们需要为TestLog创建一个新的配置Logger，修改后的配置如下所示：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Root level="info">
      <AppenderRef ref="Console"/>
    </Root>
    <Logger name="com.AnotherLog" level="trace" additivity="false">
      <AppenderRef ref="Console"/>
    </Logger>
  </Loggers>
</Configuration>
```
此时的输出结果就变为：
```
22:21:31.685 [main] INFO  com.TestLog - start TestLog!
22:21:31.690 [main] INFO  com.AnotherLog - enter AnotherLog!
22:21:31.690 [main] TRACE com.AnotherLog - you call test method!
22:21:31.690 [main] INFO  com.AnotherLog - exit AnotherLog!
22:21:31.690 [main] INFO  com.TestLog - exit TestLog!
```