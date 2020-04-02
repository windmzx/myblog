---
title: python问题记录
url: 204.html
id: 204
categories:
  - 计算机杂论
date: 2018-11-26 15:55:19
tags:
---

在这里放一些写代码中遇到的Python问题

## 'module' object has no attribute 'SSL\_ST\_INIT'
------------------------------------------------

google得到信息是python的ssl包版本不对，需要升级版本。然而在这种情况下，你无论执行什么命令都会导致这个错误，因此不能卸载掉原来的版本安装新的，需要手动处理。
```
    rm -rf /usr/lib/python2.7/dist-packages/OpenSSL
    rm -rf /usr/lib/python2.7/dist-packages/pyOpenSSL-0.15.1.egg-info
    sudo pip install pyopenssl
```
  [stackoverflow](https://stackoverflow.com/questions/45188413/python-pip-install-is-failing-with-attributeerror-module-object-has-no-att/45291334#45291334?newreg=12af3db87aad43459735cbd7b3854604)
<!--more-->
## utf8编码问题


有的时候遇到# coding: utf-8都解决不了的编码问题，可以用下面的方法解决
```
    # coding: utf-8
    import sys
    reload(sys)
    sys.setdefaultencoding('utf8')
```
## python requirement
------------------

为了当你想要分享或开源一个python项目的时候，需要生成`requirements.txt`文件，以便别人能快速的安装项目所需的依赖
```
    pip freeze > requirements.
```
## 安装cudamat的遇到的问题
---------------

有的时候安装了cuda之后再安装cudamat用于python中的矩阵加速，提示你nvcc不存在，可以用下面的方式解决
```
    sudo env "PATH=$PATH" python setup.py install
```
python快速多进程
-----------

众所周知Python是默认只能进行单个线程的进行计算，即使你手动开启了多线程，由于全局解释器锁（GIL）的问题，程序也只能使用一个cpu核心进行运行。对于IO密集型的程序还有一定的加速作用，面对计算密集型的任务来说，多线程就无能为力了。因为出现了一个包Multiprocessing可以帮你快速的实现多线程。 我们可以使用以下的代码模板来快速实现for 循环
```
    from multiprocessing import Pool
    def computefun(object):
        # do compute
    if __name__ == '__main__':
        pool = Pool(60)
        pool.map(loadats, lines)
```
Pool的参数是创建线程的数量，如是计算密集型的任务，尽量将数量设置为和你的cpu核数一样多，如果是I/O密集型则可以适当地增加数量。对于快速完成任务有很大的帮助。

not utf8 encoded saving disable
-----------------
使用jupyternotebook写了个爬虫，将数据存到文件中之后，用jupyter打开失败，发现提示`not utf8 encoded saving disable`这个错误，搜索后发现这个是因为代码里使用的是pickle进行保存，存的不是纯文本文件，所以直接打开或者是用jupyter都不能正常阅读，使用pickle进行load之后即可正常使用。
