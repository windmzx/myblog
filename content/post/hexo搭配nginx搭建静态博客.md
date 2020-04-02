---
title: hexo搭配nginx搭建静态博客
url: 103.html
id: 103
categories:
  - 计算机杂论
date: 2018-04-04 21:18:23
tags:
 - blog
---

> 这是一次很崎岖的经历，网上的很多教程都是用本地的hexo和github的服务来搭建。但是git你懂得，在天朝很有可能被某墙屏蔽掉，还有的教程是gitcafe或者是coding的服务，但是gitcafe已经被coding收购了，coding的项目演示服务是收费的，一码币需要50RMB，对于我等穷学生来说，有点承受不起。于是我选择了腾讯的服务器，认证了学生身份之后单核1G每个月只需要1元钱，绝对是福利。马云家的ECS原来是每个月10元，现在每天只能30台抢购。。.。  
> Linux用来当服务器绝对是强大的，但是各种发行版之间，软件源不同，命令不同，配置方法也不同。绝对是令人蛋疼的问题。一个nginx的搭建我就踩了很多坑。。。

# 安装过程：


环境：Centos 7.1 x64
<!--more-->
#### 1·安装node.js环境

```
curl -sL https://deb.nodesource.com/setup | bash yum install -y nodejs 
```

#### 2·安装git环境

```
yum install git-core 
```

#### 3·安装hexo

```
npm install -g hexo 
```

#### 4·新建目录

```
mkdir /www hexo init /www cd /www npm install
```

#### 5·安装nginx

```
sudo rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm 
sudo rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm 
sudo yum install nginx 
```

#### 6·nginx反向代理的设置（我的配置文件在/etc/nginx/nginx.conf）

```
server { 
	listen 80; 
	server_name yourdomian.com, www.youdomain.com; 
	location / 
	{ 
		proxy_pass http://127.0.0.1:4000/; 
		proxy_redirect off; 
		proxy_set_header X-Real-IP $remote_addr; 
		proxy_set_header X-Forwarded-For 
		$proxy_add_x_forwarded_for; 
	} 
} 
```

#### 7·开启hexo服务 
```
bash cd /www hexo s 
```

在浏览器打开你的网页看看看效果吧


# nginx的简单命令

#### 启动 ：nginx

#### 停止 ：nginx -s stop

#### 重新加载配置文件：ngixn -s reload