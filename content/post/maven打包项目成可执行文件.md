---
title: maven打包项目成可执行文件
url: 166.html
id: 166
categories:
  - 计算机杂论
date: 2018-06-13 08:50:08
tags:
  - java 
  - maven
---

通常情况下我们使用ide写的项目如果想打包成可执行文件的话，使用ide的导出功能就行，如果我们的项目有一些依赖，那么就不能依赖ide的功能实现。 使用maven打包，在pom.xml project节点下加入以下配置
```

     <build>
            <plugins>
    
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-assembly-plugin</artifactId>
                    <version>2.5.4</version>
                    <configuration>
                        <archive>
                            <manifest>
                                <mainClass>Mainclass</mainClass>
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
其中`<descriptorRef>jar-with-dependencies</descriptorRef>`是强制规定的，只有这样才能将依赖也打包进jar 
ref： 
https://blog.csdn.net/u011955252/article/details/78927175 
https://www.cnblogs.com/f-zhao/p/6929814.html