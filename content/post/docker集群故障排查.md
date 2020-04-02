---
title: docker集群故障排查
url: 139.html
id: 139
categories:
  - 计算机杂论
date: 2018-04-08 21:29:37
tags:
- docker
---

# swarm集群


本来是想检查一下集群的安全状态，集群里挂载了一个ssr作为代理，想查阅一下日志。没想到docker集群挂了，甚至有一个刀片机器挂掉了，没办法只能叫厂商来修。 集群状态是所有的节点都不能执行管理命令。使用`docker node ls`无法查看节点身份直接返回错误![](https://img.0xaa.top/TIM图片20180513193220.png) 尝试几次之后判断无法解决。使用命令`docker swarm leave -force`强制离开集群，并重建集群。 此时时间到了晚上，第二天继续维护的时候，发现管理节点依然不能管理报错如下 ![](https://img.0xaa.top/TIM图片20180513193748.png) 后来发现是node节点身份发生了变化，登陆节点失去了管理员节点身份。ssh到另一台机器上之后重新生成一个token就能成功加入了。 至此docker swarm解决完毕。
<!--more-->
# 防火墙


楼下的高性能机器是处于校园网的环境当中，也就是所有的吉大校园网用户都能连接到，因此安全是一个很大的问题。之前就有密码被攻破被拿来计算的事情发生。而且做实验的数据库也需要暴露端口到网络中，数据库的安全也是一个大问题。想要安全，又不想每开一个服务都通过ss链接到swarm网络中的代理这么麻烦，因此我们决定采用防火墙的方法。 目标是实验室的所有主机都能连接到集群中，集群之间能互相连通，可以直接暴露服务的端口而不需要考虑安全问题。因为实验室有一个固定IP，因此这件事就变得非常简单。 步骤如下 在各个节点上关闭 firewalld 服务：
```
    systemctl stop firewalld.service
    systemctl disable firewalld.service
```
安装 iptables 服务：
```
    yum install iptables-services -y
```
对 INPUT 链添加规则：
```
    iptables -I INPUT -s 59.72.109.166 -j ACCEPT
    iptables -I INPUT 2 -j DROP
    iptables -I INPUT -s 10.60.111.72 -j ACCEPT
    iptables -I INPUT -s 10.60.111.73 -j ACCEPT
    iptables -I INPUT -s 10.60.111.74 -j ACCEPT
    iptables -I INPUT -s 10.60.111.75 -j ACCEPT
    iptables -I INPUT -s 10.60.111.76 -j ACCEPT
    iptables -I INPUT -s 10.60.111.77 -j ACCEPT
    iptables -I INPUT -s 10.60.111.78 -j ACCEPT
    iptables -I INPUT -s 10.60.111.79 -j ACCEPT
    iptables -I INPUT -s 10.60.111.81 -j ACCEPT
    iptables -I INPUT -i lo -j ACCEPT
    iptables -I INPUT -p icmp -j ACCEPT
    iptables -I INPUT -p udp -s 10.10.10.10 -j ACCEPT //允许DNS解析
```
然后保存到文件里：
```
    iptables-save > /etc/sysconfig/iptables
```
经过验证实验室的机器可以直接连接到集群，通过校园网无线的动态IP或者是寝室的IP均无法连接到集群。