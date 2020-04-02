---
title: HEXO的部署-使用git来更新你的博客
url: 193.html
id: 193
categories:
  - 计算机杂论
date: 2018-09-13 20:34:18
tags:
  - blog
---

服务器端
----

### 建立git仓库

在服务器建立建立一个文件夹blog来存放文件，在blog下新建一个文件夹data.git作为git仓库。-bare是建立一个裸仓库，既只允许远程提交。
```
    mkdir data.git
    cd data.git
    git init --bare
```
<!--more-->
### 自动更新网站文件

原理是就git的钩子，每当发生推送等动作的时候，git就会自动执行你事先设置好的脚本。 创建一个文件夹来存放网站文件,由于在本地push之后，更改的是只是.git仓库里面的内容，并没有具体的文件存储在工作空间里。所以我们使用钩子进行checkout，来还原网页文件，以便于nginx进行显示。 新建一个文件hooks/post-receive输入以下内容
```
    #!/bin/sh
    export GIT_WORK_TREE=*/web 
    git checkout -f
```
  因为是bare仓库，无法直接执行checkout操作，因此我们需要制定一个环境变量`GIT_WORK_TREE`，否正的话，git不会允许你的操作 添加可执行权限  
```
    chmod +x post-receive
```

  这里有个问题是你使用hexo d将博客部署到服务器之后，刷新检出目录会发现里面空空如也，多次检查权限之后，仍然不能成功的部署。仔细检查控制台输出之后发现，提示一行小字`nothing to commit, working tree clean`猜测会不会是没有更改，导致部署不上去。对文章进行小修改之后，上上传发现已经能成功的将public文件夹部署到服务器。

### 本地端

我们新建立一个文件夹，命名为web然后在这个文件夹搭建hexo环境。
```
    hexo init web/
    cd web
    npm install
```    

npm在国内真的是非常慢，同样都是国外的pip docker，简直慢的不能忍受，还是采用阿里提供的镜像服务吧。编辑好文章后利用git更新就好。 每次完成写作之后，使用 hexo d -g就可以完成更新到服务器。

安全问题
----

为了保证安全，我们需要使用一个独立的git用户，并且禁止他的ssh登录权限，避免加入sudo组，防止其提权。