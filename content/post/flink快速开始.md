---
title: flink快速开始
date: 2019-04-13 22:29:58
tags:
 - Flink
 - 分布式
 - 流计算
---

> Apache Flink作为新一代的分布式处理引擎，用于对无界和有界数据进行状态计算。根据官方文档，flink可以支持有界和无界数据，即批处理和流处理，并将有界数据当做特殊的无界处理。根据官方文档，Flink具有更好的性能，更低的处理延时，相比storm更轻量的确认机制。

> 为了学习，首先我们就要自己搭建一套环境学习flink。官网的教程十分简单。。。自己摸索之后写下这个教程。
>
<!--more-->

 # 一 基础环境搭建下载

 Flink是基于Scala编写的一套框架，因为首先就需要我们安装Java和Scala环境。注意Java需要1.8版本以上。

 [Scala下载地址](https://www.scala-lang.org/download/all.html)。[Flink下载地址](https://flink.apache.org/zh/downloads.html) 

 注意Scala和Flink存在版本对应关系，为了不必要的麻烦，我们尽量选择稳定且对应的版本下载。本文中Java为1.8版本，Scala为2.11，Flink版本为1.8.0版本。

将Flink和Scala解压缩到/opt目录下，并将/bin加入PATH中。

执行`./bin/start-cluster.sh `启动Flink，这里和官网教程有所不同， 你会发现并没有Taskmanager。 需要执行`./bin/taskmanager.sh start` 启动Taskmanager，和官网教程一样的效果。

 ![](https://img.0xaa.top/20190414124534.png) 



 # 二 测试代码编写

 官网上给出了一个简单的wordcount的demo，但是这个教程非常简单，对于没有接触的人，不好搭建出代码环境。

 首先我们在Idea里新建一个空白的项目，存储包括以后的所有学习代码。然后新建一个java module，并添加maven支持。

 ![](https://img.0xaa.top/20190413213717.png)

 在pom.xml里添加如下依赖

 ```xml
     <dependencies>
         <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-core -->
         <dependency>
             <groupId>org.apache.flink</groupId>
             <artifactId>flink-core</artifactId>
             <version>1.8.0</version>
         </dependency>
         <dependency>
             <groupId>org.apache.flink</groupId>
             <artifactId>flink-streaming-java_2.12</artifactId>
             <version>1.8.0</version>
             <scope>compile</scope>
         </dependency>
     </dependencies>
 ```

 新建一个java类，将实例代码粘贴进去

 ```java
 import org.apache.flink.api.common.functions.FlatMapFunction;
 import org.apache.flink.api.common.functions.ReduceFunction;
 
 import org.apache.flink.api.java.utils.ParameterTool;
 import org.apache.flink.streaming.api.datastream.DataStream;
 import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
 import org.apache.flink.streaming.api.windowing.time.Time;
 import org.apache.flink.util.Collector;
 
 public class SocketWindowWordCount {
 
     public static void main(String[] args) throws Exception {
 
         // the port to connect to
         final int port=9999;
 
 
         // get the execution environment
         final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
 
         // get input data by connecting to the socket
         DataStream<String> text = env.socketTextStream("localhost", port, "\n");
 
         // parse the data, group it, window it, and aggregate the counts
         DataStream<WordWithCount> windowCounts = text
                 .flatMap(new FlatMapFunction<String, WordWithCount>() {
                     @Override
                     public void flatMap(String value, Collector<WordWithCount> out) {
                         for (String word : value.split("\\s")) {
                             out.collect(new WordWithCount(word, 1L));
                         }
                     }
                 })
                 .keyBy("word")
                 .timeWindow(Time.seconds(5), Time.seconds(1))
                 .reduce(new ReduceFunction<WordWithCount>() {
                     @Override
                     public WordWithCount reduce(WordWithCount a, WordWithCount b) {
                         return new WordWithCount(a.word, a.count + b.count);
                     }
                 });
 
         // print the results with a single thread, rather than in parallel
         windowCounts.print().setParallelism(1);
 
         env.execute("Socket Window WordCount");
     }
 
     // Data type for words with count
     public static class WordWithCount {
 
         public String word;
         public long count;
 
         public WordWithCount() {}
 
         public WordWithCount(String word, long count) {
             this.word = word;
             this.count = count;
         }
 
         @Override
         public String toString() {
             return word + " : " + count;
         }
     }
 }	
 ```

 使用Idea导包功能，导入所需的类，注意Time类不要导入错误。

 # 三 Netcat 下载安装

阅读实例代码，我们知道程序是监听一个websocket端口，并且对每一行进行分割，然后计算单位时间片内的单词总数。因此我们需要一个调试工具，netcat。

[windows版下载](https://eternallybored.org/misc/netcat/netcat-win32-1.12.zip)

[Linux版本下载](http://netcat.sourceforge.net/)

下载之后，将路径加入到PATH中。我们启动两个终端测试一下。

`nc -lL -p 9999` 
`nc localhost 9999`
在一个窗口里键入内容，可以在其他窗口看到。
![](https://img.0xaa.top/20190414130528.png)

# 四 运行测试

像storm一样，我们可以使用本地环境进行测试，或者提交到集群上进行测试。首先我们介绍本地运行方法。

## 本地环境测试

这种方法比较简单，我们在代码里设置监听本地的端口，首先将nc服务器启动，然后执行实例代码的main函数就可以运行代码。注意默认nc服务器只接受一个客户端连接。

![](https://img.0xaa.top/20190414131536.png)



我们可以看到程序计算的是单位时间片内的单词的数量。

## 提交到集群测试

想要把程序提交到集群首先我们就需要将项目打包，打包的问题就是依赖的问题。Flink的集群为我们提供了运行时环境，因为在这个demo里，我们没有使用额外的依赖，因此我们只需要将我们写的代码打包进jar就可以。

### idea导出

idea自带打包成jar的功能，打开项目设置，点击Artifacs，选择新建一个jar配置

![](https://img.0xaa.top/20190414144758.png)

![](https://img.0xaa.top/20190414144812.png)

然后添加Module Output， 选择我们的module.

![](https://img.0xaa.top/20190414145925.png)

此外可以指定输出的目录，以及默认的MainClass，注意啊需要有meta-inf文件帮我们指定mainclass或者在提交jar到集群的时候手动指定mainclass。

### maven打包

当我们需要第三方各种依赖的时候，idea自带的导出功能就显得不够强大，我们可以使用maven的配置文件帮我们自动化打包jar。在maven中添加以下节点

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                         <archive>
                            <manifest>
                                <mainClass>com.mzx.SocketWindowWordCount</mainClass>
                            </manifest>
                        </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

在idea里侧边点击package按钮或者使用`mvn package`命令都可以进行打包。

### 启动任务

将jar包提交到服务器之后，我们就可以启动任务了。使用`./bin/flink run  MyFlink.jar`就可以启动脚本。然后我们在服务器上也启动一个netcat服务器，输入文字之后，我们发现flink并没有输出，这是因为stdout没有定向到屏幕，而是输出到文件里。我们查看log下的.out文件就可以发现任务计算出的单词的个数。

![](https://img.0xaa.top/20190414133728.png)

### 补充

在之前的maven打包里，我们只对自己的文件进行了打包，而所需要的依赖由Flink环境为我们提供。当我们使用其他的依赖的时候，就需要我们将依赖也打包进jar里。这个时候可以使用如下的maven配置。

```

```

```xml
<build>
    <plugins>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>3.1.1</version>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>com.mzx.SocketWindowWordCount</mainClass>
                    </manifest>
                </archive>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

    </plugins>
</build>
```

EOF