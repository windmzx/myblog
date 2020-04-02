---
title: Docker使用速记
url: 183.html
id: 183
categories:
  - 计算机杂论
date: 2018-07-26 09:19:01
tags:
- docker
---

docker作为最近新出现的技术，一下子席卷了全球的开发者社区，无论是科研或者是商业应用，其都获得了迅猛的发展。实验室搭建了docker-swarm集群，并部署了storm、haddop、web、git托管等一系列的服务。 在docke使用过程中遇到一些问题，记录在下面解决一下。
<!--more-->
## Docker 给运行中的容器添加映射端口


很多时候我们使用docker run 命令启动容器的时候忘记了加了端口映射，此时又在容器中执行了一部分指令，重启会导致浪费之前的工作，因此我们可以给运行中的容器添加端口映射。docker使用iptables进行网络的隔离，在iptables的配置中，DOCKER链保存了各容器的网络配置，在新建容器的时候，docker会自动修改iptables规则来实现网络的隔离与通信。因此我们也可以手动的修改iptables来实现端口映射。
首先获得容器的ip地址
```
    docker inspect `container_name` | grep IPAddress
```
在DOCKER链中添加一条路由规则
```
    iptables -t nat -A DOCKER -p tcp --dport 37221 -j DNAT --to-destination ip:port
```
其中dport是外部端口，命令尾部的是容器的端口和容器内端口

## Docker 与防火墙问题


由于docker使用iptables管理网络，使用-p进行端口映射时，docker会在创建一个DCOKER链 并进行NAT转换 将宿主机的端口映射到私有网络中的容器的端口上 并将这个链插入在FORWARD链的头部（重启docker服务会自动恢复）。在这个过程中，数据包走的是FORWARD链。因此对INPUT的链的操作不会影响docker服务的访问。

解决方案：

根据官网的文档提示 https://docs.docker.com/network/iptables/#restrict-connections-to-the-docker-daemon docker在升级之后会在iptables里创建一个DOCKER-USER 链。根据官网所说在创建任何规则之前加载用户的自定义设置
```
    iptables -I DOCKER-USER  ! -i  em1 -s 59.72.*.* -j DROP
```
我们使用上面的命令将所需要访问的ip加入进去就可以，其中em1对应于外网网卡名字