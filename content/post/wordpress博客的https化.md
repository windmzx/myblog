---
title: wordpress博客的https化
url: 125.html
id: 125
categories:
  - 未分类
  - 计算机杂论
date: 2018-04-06 23:04:21
tags:
 - blog
---

随着互联网的发展，https逐渐登上了历史的舞台。https的好处有很多，例如，可以避免isp的广告植入，更好的安全性，除了用户自己，别人都只能看见通信的头信息，而看不到具体的内容，避免了一些隐私的泄露。 谷歌对于推进https是非常热心的，最近的谷歌浏览器对于全站的https站点直接标上了安全两个字。并且对于https的站点，在检索的时候会提高网站的权重，对于seo也是一个好处。 https就必然涉及到ssl证书，自己签发一个显然是不可以的，很明显你不值得谷歌信任你。强如铁老大，让用户自己装证书的狠角色现在也买了证书，甚至cnnic都用了别人的证书。之前有试用过好几款免费的证书，很不幸他们都因为违规签发GitHub或者谷歌的证书，被谷歌给移除了信任列表，真是不作死就不会死啊。 现在免费的证书Let's Encrypt比较流行。但是有个问题就是Let's Encrypt的有效期只有90天，需要自己签约，但是万能的github有人做出了一键脚本，能完成验证和自动更新。[项目地址](https://github.com/Neilpang/acme.sh)

<!--more-->
# 证书的申请
-------

完全使用这个脚本就可以完成一键域名验证以及申请，并且会帮你自动续期

# nginx的配置
----------

附docker-compose配置如下
```
    version: '2'
    services:
    mysql:
    image: mysql:5.6
    restart: always
    environment:
    MYSQL_ROOT_PASSWORD: demo
    volumes:
    - /root/wp-mysql:/var/lib/mysql
    wordpress:
    image: wordpress
    restart: always
    ports:
    - 8001:80
    environment:
    WORDPRESS_DB_PASSWORD: demo
    volumes:
    - /root/wp-file:/var/www/html
    hpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
    - 8188:80
    environment:
    PMA_HOST: mysql
```
  我的博客是在docker上跑了一个wordpress和mysql数据库。只暴露了wordpress的端口，php和mysql的通信采用docker自定义网络实现。还挂了同学的一个站点，因此需要一个nginx转发一下。nginx几乎是现在最流行的代理服务器，利用反代功能，可以实现同端口部署多个网站，并且还可以实现负载均衡等等功能。 nginx之需要一个配置文件就可以轻松的代理我的网站。配置文件如下，首先我们同时开启80，443两个端口到wordpress，以便调试。
```
    server {
            listen       443;
            server_name  0xaa.top;
    	ssl                  on;
            ssl_certificate      /etc/nginx/ssl/fullchain.cer;
    	ssl_certificate_key  /etc/nginx/ssl/0xaa.top.key;
            ssl_session_timeout  5m;
            location / {
     	    proxy_set_header X-Forwarded-Proto https;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
    	    proxy_pass  http://localhost:8001;
            }
    }
    
    
    server {
            listen       80;
            server_name  0xaa.top;
            location / {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
                proxy_pass  http://localhost:8001;
            }
    }
```

 

# wordpress程序的修改


第一步我们首先把站点的http改为https ![](https://img.0xaa.top/TIM截图20180407131827-300x46.png) 直接修改即可。 第二步，要把之前上传的图片等文件链接改为https，网上的方法有很多，修改主题等，但是一想既然已经https化，就不能再倒回去，还不如一个sql来的方便。 登陆到数据库，执行以下命令
```
    update wp_posts set post_content = replace(post_content, 'http://0xaa.top','https://0xaa.top');
```
这个时候本应该可以的，但是调试的时候遇到了两个大坑。

- 没有修改wordpress的时候，只用nginx代理到https时，所有的静态资源都不能访问。百般思索才发现是谷歌的安全策略问题，![](https://img.0xaa.top/TIM图片20180407133333.png)允许就可以了。
-  反复的重定向，导致网页完全没有办法访问，直接被浏览器拒绝。经同学提醒发现，是因为nginx配置有问题导致，wp一直以为自己收到的是http请求，不断的重定向，解决办法就是修改一下配置文件
    
        proxy_set_header X-Forwarded-Proto https;
        
    
    加入这一行就可以了。

这个时候访问，发现浏览没有问题，但是小绿锁并没有出现。。。内心千万只羊驼。打开F12检查发现，是我的主题的组件里面的链接是写死的。去后台修改一下，设置成https链接就可以了。

# 301跳转


现在http链接已经完美的支持了，是时候让整个网站只使用https了。我们修改nginx配置文件，让http请求重定向到https。

    server {
            listen       80;
            server_name  0xaa.top;
            return 301 https://0xaa.top$request_uri;
    }

检查一下发现，ok。 ![](https://img.0xaa.top/TIM截图20180407134251.png)