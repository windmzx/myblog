---
title: Neo4j使用总结
url: 176.html
id: 176
categories:
  - 计算机杂论
date: 2018-07-19 13:15:44
tags:
---

实验室研究的方向是异质信息网络，因此图数据库是一个非常常用的工具。这篇文章记录 一些问题便于速查。

### py2neo的版本

为了操作Neo4j数据库，采用了python语言的一个包，py2neo。 py2neo比较常用的有三个大版本。v2,v3,v4每个版本之间的差异比较大。其中v3有个Grapg.data()函数，可以执行原生的cypher语句并返回一个python原生的结果集。但是如果你不注意pip安装的时候指定脚本就会发送找不到data()函数的错误。neo4j graph has no attribute data 这个时候需要指定安装版本。
```
    pip3 install "py2neo>3.0,<4.0"
```
<!--more-->
### 强制删除节点


neo4j数据库为了保证数据的安全，默认情况下不能删除带有边的节点，防止边上的信息被删除。如果你确定点和边的信息是不需要的 可以在delete node之前加上detach关键字，进行强制删除。

删除重复的关系


### 删除重复关系
如果你插入关系的时候没有对关系的唯一性进行检查，会导致两个节点之间多次插入相同的关系。当使用到边的权重的时候会导致数据错误，为了删除多余的关系，可以使用下面的命令。
```
    MATCH (a)-[r:target_rel]->(b)
    WITH a, b, TAIL (COLLECT (r)) as rr
    WHERE size(rr)>0
    FOREACH (r IN rr | DELETE r)
```

### 数据安装过程中的配置问题
由于neo4j在工作的过程中多个协议使用多个端口，在一个机器上运行多个实例，非常容易发生端口混乱的情况。尤其是在dockerswarm集群的时候，当你指定了集群网络之后，端口会冲突，因此在应用的过程中不使用swarm部署服务或者在配置中修改默认的端口（虽然这样改了之后仍然会发生有多个实例的情况下，数据会串库）因此建议使用原生的方式部署。这时候需要修改它的几个配置文件如下 `default_advertised_address`这个配置是为了浏览器访问的时候默认填充blot地址，否则每次手动输入比较麻烦。开放给所有的地址，安全由防火墙来保证。
```
    dbms.connectors.default_listen_address=0.0.0.0
    dbms.connectors.default_advertised_address=192.168.1.230
    # Bolt connector
    dbms.connector.bolt.enabled=true
    #dbms.connector.bolt.tls_level=OPTIONAL
    dbms.connector.bolt.listen_address=:7687
    
    # HTTP Connector. There can be zero or one HTTP connectors.
    dbms.connector.http.enabled=true
    dbms.connector.http.listen_address=:7474
    
    # HTTPS Connector. There can be zero or one HTTPS connectors.
    dbms.connector.https.enabled=true
    dbms.connector.https.listen_address=:7473
```

使用docker的情况下则添加环境变量命令如下
```
    --env=NEO4J_dbms_connector_http_listen__address=0.0.0.0:7475 \
    --env=NEO4J_dbms_connector_bolt_listen__address=0.0.0.0:7688 \
    --env=NEO4J_dbms_connector_https.listen__address=0.0.0.0:7472 \
```