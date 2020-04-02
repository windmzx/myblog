---
title: 路由器上进行drcom认证
url: 101.html
id: 101
categories:
  - 智能硬件
date: 2018-04-04 21:15:56
tags: 
  - hardware
---

> 吉大的校园网说实话还是很不错的，每个月20，常用资源分分钟上100m，还有教育网ipv6的资源。但是手机和电脑只能同时在线一个（更新：现在能同时在线了但是无线网也还是水的一比，基本在不能用的水平），这个教程的意义在于让使用路由器实现手机和电脑同时在线。目的绝不是共用一个账号来逃避费用，这点学校是禁止的。

1.用winscp软件把zlib\_1.2.7-1\_ralink.ipk python\_2.7.3-2\_ralink.ipk python-mini\_2.7.3-2\_ralink.ipk三个ipk包上传到/tmp目录下面 下载链接 [zlib\_1.2.7-1\_ralink.ipk](http://downloads.openwrt.org.cn/PandoraBox/ralink/mt7620_old/packages/zlib_1.2.7-1_ralink.ipk) [python\_2.7.3-2\_ralink](http://downloads.openwrt.org.cn/PandoraBox/ralink/mt7620_old/packages/python_2.7.3-2_ralink.ipk) [python-mini\_2.7.3-2\_ralink](http://downloads.openwrt.org.cn/PandoraBox/ralink/mt7620_old/packages/python-mini_2.7.3-2_ralink.ipk) 2.ssh连接路由器  
执行以下命令 \[code lang=bash\] cd /tmp opkg install zlib* opkg install python* \[/code\] 等待安装完成 3配置drcom脚本 [Gitgub](https://github.com/drcoms/jlu-drcom-client)下载文件，把for-OpenWRT/中的内容解压出来修改drcom.conf 中的帐户名 密码 绑定的mac地址 把文件用winscp上传到/tmp 目录下 \[code lang=bash\] chmod 777 ./install.sh ./install.sh \[/code\] 之后切换到/usr/bin 目录 \[code lang=bash\] cd /usr/bin python wired.py \[/code\] 这时候就可以验证登录,如果不想每次都需要手动启动还可以加到开机启动 \[code lang=bash\] cd /tec vi rc.local \[/code\] 在exit 0 之前加上 \[code lang=bash\] python /usr/bin/wired.py \[/code\]