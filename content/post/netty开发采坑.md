---
title: "Netty开发采坑"
date: 2020-05-11T18:37:08+08:00
draft: true
---

最近在学习Netty，一个高性能的I/O框架，在很多中间件中都在使用。想用netty实现一个聊天程序，在此期间遇到了很多问题。记录一下。

### ctx.writeAndFlush() 和ctx.channle().writeAndFlush()的区别

ctx.writeAndFlush()是从handler链寻找下一个outboud出口处理，而ctx.channle().writeAndFlush()会从handler链的最后一个开始传递，找到outboundhandler处理。

按照正确的顺序添加handler可以避免无用的判断。

### 长度分割

使用LengthFieldBasedFrameDecoder分割字节流组成包，发现无法传递消息到decode过程，将其注释之后decode出错，不能正确获取数据的长度。仔细检查发现是在之前的代码里服务器没有严格按照协议传输一串字符串过来，导致无法encode或者encode失败。



