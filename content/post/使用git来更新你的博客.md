---
title: 使用git来更新你的博客
url: 98.html
id: 98
categories:
  - 计算机杂论
date: 2018-04-04 21:11:50
tags:
---

> 上一篇文章中，我们用hexo和nginx搭建了一个静态博客，但是我们怎么更新博客呢？每次都用ftp登陆服务器之后，手动的把文章传上去？每次修改文章之后也要上传一次？其实不用，我们可以用git这个强大的工具来完成。

**服务器端**
--------

### 建立git仓库

在服务器建立建立一个文件夹blog来存放文件，在blog下新建一个文件夹data.git作为git仓库

    mkdir data.git
    cd data.git
    git init --bare
<!--more-->
 

### 自动更新网站文件

原理是就git的钩子，每当发生推送等动作的时候，git就会自动执行你事先设置好的脚本。 创建一个文件夹来存放网站文件,由于在本地push之后，更改的是只是.git仓库里面的内容，并没有具体的文件存储在工作空间里。所以我们使用钩子进行checkout，来还原网页文件，以便于nginx进行显示。 `mkdir web #新建一个文件hooks/post-receive输入以下内容` 因为是bare仓库，无法直接执行checkout操作，因此我们需要制定一个环境变量`GIT_WORK_TREE`

    #!/bin/sh
    export GIT_WORK_TREE=/web 
    git checkout -f
    

`添加可执行权限`

    chmod +x post-receive

   

**本地端**
-------

我们新建立一个文件夹，命名为web然后在这个文件夹搭建hexo环境。

    mkdir web
    hexo init web/
    cd web
    npm installl

npm再国内真的是非常慢，同样都是国外的pip docker，简直慢的不能忍受，还是采用阿里的代理吧 编辑好文章后利用git更新就好。

    git add.
    git commit -m "提交的备注"
    git remote add origin git@ip:/blog/data.git#添加远程仓库
    git push origin master

origin是可以随意起名字的，但是git clone的时候默认会把远程仓库标记为origin。这样以后每次在本地写完文章执行git add git commit git push三个步骤就可以更新博客