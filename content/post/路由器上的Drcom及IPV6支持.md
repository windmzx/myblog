---
title: 路由器上的Drcom及IPV6支持
url: 115.html
id: 115
categories:
  - 智能硬件
  - 计算机杂论
date: 2018-04-05 19:41:11
tags: 
   - hardware
---

学校正在网络改造，给每个宿舍改成千M入户以及安装AP面板，提供更好的无线性能。路由器的作用越来越小，但是对于有很多台设备的同学来说，一个路由器还是很有必要的，可以组建局域网打游戏和安装NAS等。本文章在k2p Padavan 系统下进行，如果你照做不好使，可能是系统的差异。

### Dr.com认证

为了愉快的上网，江山代有才人出。各位大佬相继开发了c，java，python，c#等等版本的客户端，今天介绍一个学弟写的c版，可以交叉编译成arm版，直接运行。 [https://github.com/hyec/drcomlite](https://github.com/hyec/drcomlite) 编译之放在/usr/local/bin里面，即可运行（注意添加权限，据说师弟要发release，这样就不用自己编译了）  

### IPv6

之前使用dhcpv6的办法，现在使用桥接将wan和lan桥接在一起，然后过滤掉网桥上的v4流量，就可以实现v4走nat，v6直连局域网所有的机器。

    modprobe ip6table_mangle
    modprobe ebtable_broute
    ebtables -t broute -A BROUTING -p ! ipv6 -j DROP -i eth3
    brctl addif br0 eth3

  将以上文件写在开机脚本里就可以实现开机自动配置IPv6。 注意，以往都是用路由器克隆电脑的mac，用来支持学校的ip，mac绑定，如果是这样的情况下，你会发现执行以上命令后，电脑再也链接不了路由器，这是因为mac冲突造成的，只要修改一方的mac地址就可以修复。 学校的网总是掉线，为了断线重连，写了一个脚本，本人对于shell不甚了解，有更好的方法，欢迎指出。

    #! /bin/bash
    #检测网络连接
    ping -c 1 114.114.114.114 > /dev/null 2>&1
    if [ $? -eq 0 ];then
       echo 检测网络正常
    else
       echo 检测网络连接异常
       killall drcom &&drcom
    fi