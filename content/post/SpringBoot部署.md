---
title: "SpringBoot部署"
date: 2020-05-20T14:39:23+08:00
---

> 完成了一个springboot的项目，打算部署到公网上，写下此文，记录过程中的尝试 
> Dome地址 [问答网站](http://wenda.0xaa.top)
## 打包
```shell
mvn package -Dmaven.test.skip=true
```
把项目打包成jar包，我们可以使用内置的tomcat来部署，只需要运行`java -jar app.jar`就可以启动项目

将打包完的jar包拷贝到服务器上，使用一下dockerfile打包成docker镜像
```
FROM java:8
VOLUME /tmp
ADD wenda.jar wenda.jar
EXPOSE 8080
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/wenda.jar"]

```
## 部署
### 开启mysql

```shell
docker run -p 3306:3306 --name mysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root5rdx$ESZ -d mysql:5.7
```

### 开启项目
```shell
docker run -p 8080:8080 --name wenda -e MYSQL_PASSWORD=123456 -e REDIS_HOST=redis mzx/wenda
```
### 容器互联问题
1. msql并不安全
2. 数据库的主机名随着部署环境的变化，需要重新修改打包

在docker的以前的版本中，我们可以使用`--link`选项将我们的应用镜像和容器镜像绑定到一个网桥上，并由Docker引擎刷新hosts。后来的版本推荐使用用户自定义网络。
我们创建一个网桥属性的网络，将后来的容器都部署到这个网络中。

```shell
docker network create -d bridge blog
```
此时，数据库的启动脚本变为
```shell
docker run  --name mysql --network=blog *** mysql
```
我们的容器也同样使用`--network`属性绑定网络。
这个时候我们就可以在Spring的配置文件中使用mysql等主机名连接数据库，但是这又和我们本地的环境相互冲突，在调试和打包的时候需要修改配置。我们可以使用环境变量来传递数据库的主机名和密码。

```
spring.datasource.url=jdbc:mysql://${MYSQL_HOST:localhost}:3306/wenda?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=UTC&autoReconnect=true&failOverReadOnly=false
REDIS_HOST=localhost
spring.datasource.username=${MYSQL_USER:root}
spring.datasource.password=${MYSQL_PASSWORD:root}
```
其中直接在配置文件中获取环境变量就`${:}`获取，等号前面是环境变量的名字，等号后面则是不存在环境变量的时候的默认值。
在代码中则使用`@Value`注解获取。
```
@Value("${REDIS_HOST}")String redis_host;
```
就可以注入到类中，如果环境变量中没有，则去配置文件读取默认值。
在启动镜像的时候使用`-e Een:Envval`就i可以传递环境变量。
### Redis部署
我们的项目中使用了Redis，因此我们需要使用Docker部署一个Redis容器。
```shell
docker run --name some-redis --network=blog  -v /root/wenda/reidsdata/:/data -d redis redis-server --appendonly yes
```
使用`--appendonly yes`进行持久化，具体可以搜索Redis的两种持久化方式。

## 遇到的问题

### 数据库总是连接超时
在一段时间没有流量的时候，数据库总是连接超时。即使我们在数据库连接后面增加了`autoReconnect=true&failOverReadOnly=false`也会出现这个问题。我们可以调整数据库的超时时间，但是超时时间不能无限增加，可能会保存过多的连接导致性能浪费。
```
spring.datasource.test-while-idle=true
spring.datasource.time-between-eviction-runs-millis=8000
spring.datasource.min-evictable-idle-time-millis=1000
```
我们可以考对连接池进行配置，进行检测。

### 静态资源访问太慢
我用的是白嫖的阿里云主机，只有1M的带宽。大的js和图片那叫一个慢啊。
可以使用阿里云的oss来加速访问。
首先我们在阿里云oss控制台新建一个bucket。
然后将spring boot staic中的静态文件上传。
然后在nginx里如下配置

```
server {
        listen 80;
        server_name  wenda.0xaa.top;

        location / {
        # pass to aproxy
        proxy_pass http://127.0.0.1:8080;
        }

        location /styles {
        expires 24h;
        proxy_pass http://mzxwenda.oss-cn-beijing.aliyuncs.com;
        }

        location /images {
        expires 24h;
        proxy_pass http://mzxwenda.oss-cn-beijing.aliyuncs.com;
        }
        location /scripts {
        expires 24h;
        proxy_pass http://mzxwenda.oss-cn-beijing.aliyuncs.com;
        }
}
```

### 后记

#### adminer

一个非常简单轻量级的mysqlui，用于导入数据之后的非常方便