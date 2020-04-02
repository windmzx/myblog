---
title: 通过vagrant快速部署flink集群
date: 2019-09-06 19:31:40
tags: 
- 云计算
---

flink作为新一代的流式计算平台，为了学习这个平台我们首先肯定要在自己的机器上搭建一个尝试一下。每次搭建环境都是一个噩梦，幸而有了docker和vagrant。docker倾向于快速的部署，避免在生产环境上的依赖问题。而vagrant更倾向于在开发过程中团队共享相同的开发环境。

vagrant其实是基于VirtualBox的，将vbox的api进行了进一步的封装，便于我们使用，因此想使用vagrant首先要安装vbox，在安装vagrant。

在[github](https://github.com/zhenzhongxu/vagrant-flink)上我找到了前人已经写好的脚本，因为年代比较久远，有些版本和依赖已经不受支持，因此我对脚本进行了一下更新。
[更新版](https://github.com/windmzx/vagrant-flink)

- 将java的安装改为去甲骨文网站下载二进制文件安装。
- 修改了环境变量的配置，避免因为flink的bug读不到JAVA_HOME,无法启动。
- 修改了内存的配置，避免因为内存原因启动失败。
- 修改支持更新版本的flink。



注意：在Vagrantfile里可以配置taskmanager的数量
vm的内存一定要大于flink配置文件(setup-flink.sh)中设置的java虚拟机的内存大小。

