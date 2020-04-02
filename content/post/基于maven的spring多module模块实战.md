---
title: 基于maven的spring多module模块实战
url: 159.html
id: 159
categories:
  - Code
  - Java
date: 2018-05-30 20:09:07
tags:
---

写毕业论文之后，博客文章题目起得的都学术化了。。。 当一个项目越来越大，我们需要划分模块避免一个项目变得臃肿。目前网上的例子都比较老，几经尝试才成功，因此记录下自己过程以便回忆，也是给其他朋友的一个参考。 技术栈 IDEA+maven+sping boot 话不多说，直接上代码。

1、建立父项目

使用root来定义我们的父项目 ![](https://img.0xaa.top/TIM截图20180530195808.png) 建立好之后，删掉src目录，只留下pom.xml。 修改pom:

1.  把packaging方式由jar改为pom
2.  如果父项目有build节点的话，删除掉

2分别建立几个module

<!--more-->
命名为model，dao，service，web，来分别保存每一层的代码。 ![](https://img.0xaa.top/TIM截图20180530195951.png) 建好后的效果 ![](https://img.0xaa.top/TIM截图20180530200816.png)

3.为整个工程添加依赖


因为所有的module都是依赖于子项目的，因此我们可以直接把以来写进父项目。修改父项目的pom如下
```
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    
    	<groupId>com.mzx</groupId>
    	<artifactId>ntms</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    	<modules>
    		<module>model</module>
    		<module>dao</module>
    		<module>service</module>
    		<module>web</module>
    	</modules>
    	<packaging>pom</packaging>
    
    	<name>ntms</name>
    	<description>Demo project for Spring Boot</description>
    
    	<parent>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-starter-parent</artifactId>
    		<version>2.0.2.RELEASE</version>
    		<relativePath/> <!-- lookup parent from repository -->
    	</parent>
    
    	<properties>
    		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    		<java.version>1.8</java.version>
    	</properties>
    
    
    <dependencies>
    
    	<dependency>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-starter-web</artifactId>
    	</dependency>
    
    	<dependency>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-starter-data-jpa</artifactId>
    	</dependency>
    
    	<dependency>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-devtools</artifactId>
    		<scope>runtime</scope>
    	</dependency>
    	<dependency>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-starter-test</artifactId>
    		<scope>test</scope>
    	</dependency>
    
    	<dependency>
    		<groupId>org.projectlombok</groupId>
    		<artifactId>lombok</artifactId>
    	</dependency>
    	<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    	<dependency>
    		<groupId>mysql</groupId>
    		<artifactId>mysql-connector-java</artifactId>
    		<version>6.0.6</version>
    	</dependency>
    
    </dependencies>
    
    </project>
```
boot节点指定了整个工程的spring版本，dependency则加入了一个springboot项目一些必要的依赖。model节点则定义了多个module

4.添加其他子项目


model项目的pom如下
```
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <parent>
            <artifactId>ntms</artifactId>
            <groupId>com.mzx</groupId>
            <version>0.0.1-SNAPSHOT</version>
        </parent>
        <modelVersion>4.0.0</modelVersion>
    
        <artifactId>model</artifactId>
        <groupId>com.mzx</groupId>
        <version>0.0.1-SNAPSHOT</version>
    
    
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    </project>
```
其中parent节点指定了父项目，并且加入了build节点，因为model项目打包之后提供给dao层调用。其中默认的定位信息不全，或者你想和父项目使用不同的报名。一定要指定好`artifactId,groupId,version 三条属性。因为在别的module也要用的到。` dao层的pom如下，与model不同的是，dao需要调用model的方法，因此他要依赖于model。
```
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <parent>
            <artifactId>ntms</artifactId>
            <groupId>com.mzx</groupId>
            <version>0.0.1-SNAPSHOT</version>
        </parent>
        <modelVersion>4.0.0</modelVersion>
    
        <artifactId>dao</artifactId>
        <groupId>com.mzx</groupId>
        <version>0.0.1-SNAPSHOT</version>
    
        <dependencies>
            <dependency>
                <artifactId>model</artifactId>
                <groupId>com.mzx</groupId>
                <version>0.0.1-SNAPSHOT</version>
            </dependency>
        </dependencies>
    
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    
    </project>
```
其他几层也依法炮制，值得注意的点就是需要添加对其他module的依赖，否则会报错。

5.Idea的设置


需要注意的一点是，虽然我们在pom里指定了各个module的依赖关系，但是IDEA并不能识别，需要我们在idea设置一下，否则代码提示上会有问题。 ![](https://img.0xaa.top/TIM图片20180530205531.png) 添加对其他项目的module的依赖即可。

6.代码Test


### moudel -Student.java
```
    @Entity
    public class Student {
        @Id
        Long id;
        String username;
        String name;
        int sex;
        int age;
        @Column
        int idcard;
    
        public Student() {
        }
    
        public Student(String username, String name, int sex, int age, int idcard) {
            this.username = username;
            this.name = name;
            this.sex = sex;
            this.age = age;
            this.idcard = idcard;
        }
    //getter and setter
    }
```
这个地方需要注意的一点是除了你需要的构造函数之外，必须新建一个无参构造函数，否则orm框架无法利用反射填充对象的属性值。

### dao-UserDao.java
```
    @Repository
    public interface UserDao extends JpaRepository<Student,Long> {
        List<Student> findByName(String name);
    }
```

dao层我们使用data-jpa,对于简单的方法，我们可以直接使用方法名即可完成sql的自动生成。

### service

演示的service，我们直接返回dao层的查询结果就可以。
```
    @Service
    public class TestService {
    
        @Autowired
        UserDao userDao;
        public List testservice(String username){
            return userDao.findByName(username);
        }
    }
    
```
### web

我们直接使用一个restcontroller返回数据的查询结果
```
    @Slf4j
    @RestController
    @RequestMapping("/")
    public class HomeController {
    
        @Autowired
        TestService service;
    
        @RequestMapping(value = "/home/{name}")
        public List<Student> home(@PathVariable("name")String name) {
            log.info("jaja");
            return service.testservice(name);
        }
    }
```
新建一个Application类启动我们的项目

```    
    @SpringBootApplication
    public class WebApplication {
        public static void main(String[] args) {
            SpringApplication.run(WebApplication.class, args);
        }
    }
```
在resources下新建一个application.yml来存储我们的配置
```
logging:
  level:
    web: info
    root: info
spring:
  datasource:
    password: 123456
    username: root
    url: jdbc:mysql://localhost:3306/ntms?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=UTC
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: update
```
启动项目之后，我们使用UI界面连接到数据库。发现jpa已经为我们新建好了数据表，我们插入一下假的数据，在浏览器下访问，可以返回结果就是正确的。至此我们的多module已经成功了。其中遇到了很多问题，我把一些较为复杂的记录在另一篇文章里。 \[embed\]https://0xaa.top/157.html\[/embed\]