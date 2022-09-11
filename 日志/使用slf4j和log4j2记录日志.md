# 使用slf4j和log4j2记录日志

## 1 入门



### 1.1 slf4j简介

slf4j是一个日志服务中间层。slf4j封装了多种日志库的接口，使用slf4j后，若是要修改程序使用的日志库，只须要将对应日志库的jar放入classpath，不须要修改任何代码。slf4j为部署时更换日志库提供了灵活便利。apache



### 1.2 HelloWorld for slf4j

下面是一个来自slf4j官网的例子。

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

这段代码展现了slf4j的基本用法。在编译和执行时，须要将slf4j-api-1.7.22.jar和slf4j-simple-1.7.22.jar加入classpath。这两个jar包能够从slf4j官网下载。

### 1.3 log4j2简介

log4j2是一个日志库。log4j2是log4j的第二版，log4j2和log4j并不兼容。目前，log4j已经中止维护。debug

### 1.4 HelloWorld for slf4j & log4j2

slf4j做为日志服务中间层，将调用方和日志库隔离开，调用方不须要知道任何日志库的细节。在部署时，只需将对应日志库的jar包加入classpath，就能够使用这个日志库。 将上面例子中的classpath稍做修改，增长下面3个jar包：log4j-slf4j-impl-2.x.jar、log4j-api-2.x.jar、log4j-core-2.x.jar，移除slf4j-simple-1.7.22.jar， 就成了slf4j和log4j的HelloWorld示例。日志

若是使用log4j2做为日志库，须要对其进行配置。log4j2的默认配置文件是classpath下的log4j2.xml。下面是一个简单的log4j2.xml文件示例。xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
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

## 2 用法

### 2.1 引入依赖包

在编译时，classpath中须要加入对象

- slf4j-api-1.7.22.jar

在运行时，classpath须要加入接口

- slf4j-api-1.7.22.jar
- log4j-slf4j-impl-2.7.jar
- log4j-api-2.7.jar
- log4j-core-2.7.jar



### 2.2 导入类

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
```

### 2.3 构造logger对象

```
Logger logger = LoggerFactory.getLogger(MyClass.class);
```



### 2.4 记录日志

```
logger.info("Hello");
logger.debug("Temperature set to {}.", t);
```



### 2.5 配置log4j2

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Root level="error">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```



## 3 slf4j说明



### 3.1 日志级别

slf4j支持如下级别的日志部署

- trace
- debug
- info
- warn
- error



## 4 log4j2说明



### 4.1 配置文件



#### 4.1.1 配置文件路径

log4j2的默认配置文件是classpath中的log4j2.xml。在启动程序时，能够经过设置参数log4j.configurationFile的方式手动指定log4j2配置文件。get

```
-Dlog4j.configurationFile=log4j2.xml
```



## 5 参考资料

- slf4j官网 [http://www.slf4j.org/](http://www.javashuo.com/link?url=http://www.slf4j.org/)
- log4j2官网 [http://logging.apache.org/log4j/2.x/](http://www.javashuo.com/link?url=http://logging.apache.org/log4j/2.x/)





附录：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="off">
    <Appenders>
        <Console name="CONSOLE" target="SYSTEM_OUT">
          <PatternLayout pattern="%d{MM-dd HH:mm:ss.SSS} %-5level %c{1.}:%L %msg %X{username} %X{operateResult} %n "/>
        </Console>
    </Appenders>


    <Loggers>
        <logger name="com.sitech.bds.serivce.impl" level="INFO" additivity="true">
            <AppenderRef ref="CONSOLE"/>
        </logger>
        <logger name="com.sitech.bds.aop" level="INFO" additivity="true">
            <AppenderRef ref="CONSOLE"/>
        </logger>
        <logger name="com.sitech.bds.job" level="INFO" additivity="true">
            <AppenderRef ref="CONSOLE"/>
        </logger>
        <logger name="com.sitech.bds.interceptor" level="INFO" additivity="false">
            <AppenderRef ref="CONSOLE"/>
        </logger>
        <logger name="com.sitech.bds.websocket" level="ALL" additivity="false">
            <AppenderRef ref="CONSOLE"/>
        </logger>
        <logger name="com.sitech.bds.websocket" level="ALL" additivity="false">
            <AppenderRef ref="CONSOLE"/>
        </logger>
<!--        <logger name="druid.sql.DataSource" level="ALL" additivity="false">-->
<!--            <AppenderRef ref="CONSOLE"/>-->
<!--        </logger>-->
<!--        <logger name="druid.sql.Connection" level="ALL" additivity="false">-->
<!--            <AppenderRef ref="CONSOLE"/>-->
<!--        </logger>-->
<!--        <logger name="org.springframework.cache" level="ALL" additivity="false">-->
<!--            <AppenderRef ref="CONSOLE"/>-->
<!--        </logger>-->
<!--        <logger name="org.springframework.orm.jpa" level="debug" additivity="true">-->
<!--            <AppenderRef ref="CONSOLE"/>-->
<!--        </logger>-->
<!--        <logger name="org.springframework.transaction" level="debug" additivity="true">-->
<!--            <AppenderRef ref="CONSOLE"/>-->
<!--        </logger>-->
<!--        <logger name="org.hibernate" level="debug" additivity="true">-->
<!--            <AppenderRef ref="CONSOLE"/>-->
<!--        </logger>-->


        <Root level="DEBUG">
            <AppenderRef ref="CONSOLE"/>
        </Root>
    </Loggers>

</Configuration>
```







































