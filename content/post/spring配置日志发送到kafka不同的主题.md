---
title: "Spring配置日志发送到Kafka不同的主题"
date: 2020-07-22T10:09:57+08:00
---

最近在用Flink做大数据分析。对网站的访问日志，用户的行为进行分析，我们采取的框架是将Spring的日志写到Kafka之中，然后Flink读取日志信息做实时的分析，并将结果写入Redis，供业务使用。

如果将是所有的日志都写进一个Kafka的topic的话，我们读取的时候消费一次要发送到不用的流中，流计算拓扑结构较为复杂。如果使用Kafka的分组机制，虽然可以每个计算流可以独立消费消息，但是对于Kafka的压力就比较大，无用的消息也会进入流中，需要我们进行过滤操作等。

因此 我们考虑在一开始就将不同的日志写入到不同的topic里。



我们采用的框架如下：

- spring-boot
- slf4j+log4j2
- kafka  

使用slf4j的原因就是因为他可以匹配不同的日志后端组件，便于我们统一维护。

在[log4j的官网](https://logging.apache.org/log4j/2.x/manual/appenders.html#KafkaAppender)上我们可以看到给出的实例代码，如何将日志写到kafka中。使用官方维护的KafkaAppenders就可以。`syncSend`参数用来指定是否同步发送。

```xml
<?xml version="1.0" encoding="UTF-8"?>
  ...
  <Appenders>
    <Kafka name="Kafka" topic="log-test">
      <PatternLayout pattern="%date %message"/>
        <Property name="bootstrap.servers">localhost:9092</Property>
    </Kafka>
  </Appenders>

```







那么如何解决不同的日志写到不同的主题里呢。我们继续查阅官网，发现有一个叫`Filter`的可以对日志进行过滤。     

```
<MarkerFilter marker="FLOW" onMatch="ACCEPT" onMismatch="DENY"/>
```

而[MarkerFilter](https://logging.apache.org/log4j/2.x/manual/filters.html#MarkerFilter)则可以对标记进行过滤。我们如果给出这个标记呢？

在[Marker](https://logging.apache.org/log4j/2.x/manual/markers.html)这一章节中，我们看到给出的Demo。

```java
import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.MarkerManager;
import java.util.Map;
 
public class MyApp {
 
    private Logger logger = LogManager.getLogger(MyApp.class.getName());
    private static final Marker SQL_MARKER = MarkerManager.getMarker("SQL");
    private static final Marker UPDATE_MARKER = MarkerManager.getMarker("SQL_UPDATE").setParents(SQL_MARKER);
    private static final Marker QUERY_MARKER = MarkerManager.getMarker("SQL_QUERY").setParents(SQL_MARKER);
 
    public String doQuery(String table) {
        logger.traceEntry();
 
        logger.debug(QUERY_MARKER, "SELECT * FROM {}", table);
 
        String result = ... 
 
        return logger.traceExit(result);
    }
}
```

我们将这段代码，copy到我们的项目之中，这中间有一个问题就是，`Marker`的位于的包是，`org.apache.logging.log4j`，而我们使用的Slf4j的log接口，类型并不匹配，我们要改为使用Slf4j提供的Marker和MarkerFactory。

```java
import org.slf4j.Marker;
import org.slf4j.MarkerFactory;

private static final Marker LOGIN_MARKER = MarkerFactory.getMarker("LOGIN");
```



这样我们就对登陆日志做好了一个标记。

```xml
     <Kafka name="Kafka1" topic="req_log" syncSend="false">
            <MarkerFilter marker="REQ" onMatch="ACCEPT" onMismatch="DENY"/>
<!--            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>-->
            <PatternLayout pattern="%d{UNIX_MILLIS},%message"/>
            <Property name="bootstrap.servers">192.168.1.175:9092,192.168.1.175:9093</Property>
        </Kafka>

        <Kafka name="Kafka2" topic="login_log" syncSend="false">
            <MarkerFilter marker="LOGIN" onMatch="ACCEPT" onMismatch="DENY"/>
<!--            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>-->
            <PatternLayout pattern="%d{UNIX_MILLIS},%message"/>
            <Property name="bootstrap.servers">192.168.1.175:9092,192.168.1.175:9093</Property>
        </Kafka>
```

回到log4j的配置文件中，我们定义好两个Appender，分别过滤自己想要的日志，并且指定发送到不同的topic中。最后在Loggers属性中引用就可以。

我们开两个消费者，查看我们的日志是否成功写入。

![结果](http://img.0xaa.top/20200722103400.png)