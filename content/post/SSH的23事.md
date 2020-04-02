---
title: SSH的23事
date: 2019-09-03 19:55:01
tags:
---

Linux作为开发者的必备利器，经常使用到。但是因其发行版众多和Xserver的效率比不上windows和mac，因此我们还是常使用虚拟机或远程开发机。

本文将记录下使用linux系统的一些简单命令或优化操作，毕竟好记性不如烂笔头。


### 不使用Xshell等工具管理远程主机

因为我们必然有不止一台主机，如何在不使用第三方管理工具的情况下快速登陆多个主机呢。一种方法便是ssh config。注意使用sshconfig需要使用密钥登陆。
1、首先在本地生成密钥对
```
ssh-keygen -t rsa
```

2、其次将密钥对拷贝到远程主机上`ssh-copy-id user@server` 可以使用-i 参数指定本地的密钥文件。

3、在`~/.ssh/conig `文件中填入主机的配置信息
```
Host 229
    Hostname 192.168.1.229
    Port 22
    User test
    IdentityFile ~/.ssh/id_rsa
```

4、在命令行中使用`ssh `加上文件中设置的别名即可一键登陆。

### SSH登陆很慢
我们的机器都在一个大局域网下，每次登陆发现速度都很慢，表现就是很长时间后才会提示你输入密码。这个是因为远程主机在尝试反向解析客户端的hostname。我们将远程主机的这个服务禁用即可。
编辑`etc/ssh/sshd_config` 将设置改为如下`UseDNS no`,`GSSAPIAuthentication no`