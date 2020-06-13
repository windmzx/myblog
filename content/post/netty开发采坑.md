---
title: "Netty开发采坑"
date: 2020-05-11T18:37:08+08:00
---

> 最近在学习Netty，一个高性能的I/O框架，在很多中间件中都在使用。想用netty实现一个聊天程序，在此期间遇到了很多问题。记录一下。

### ctx.writeAndFlush() 和ctx.channle().writeAndFlush()的区别

ctx.writeAndFlush()是从handler链向前处理，而ctx.channle().writeAndFlush()会从handler链寻找最后一个outboud出口开始处理。

按照正确的顺序添加handler可以避免无用的判断。

### 长度分割

使用LengthFieldBasedFrameDecoder分割字节流组成包，发现无法传递消息到decode过程，将其注释之后decode出错，不能正确获取数据的长度。仔细检查发现是在之前的代码里服务器没有严格按照协议传输一串字符串过来，导致encode时获取到错误的长度和偏移量。反序列化出错抛出异常，因此后续的handler无法接受到消息。

###  spring注入顺序

当我们接受到一个请求之后，我们从他的类型就可以判断出消息的类型，从而调用相应的handler处理，而不需要在漫长的handler链上进行处理。因为我们可以在逻辑上平行的handler放到map里，根据消息类型直接调用。

```java
public class IMHandler extends SimpleChannelInboundHandler<Packet> {

    private Map<Byte, SimpleChannelInboundHandler> handlerMap;

    @Autowired
    GroupCreateHandler groupCreateHandler;

    @Autowired
    GroupMessageHandler groupMessageHandler;

    @Autowired
    MessageHandler messageHandler;


    
    private IMHandler() {
        handlerMap = new HashMap<>();
        handlerMap.put(Command.CREATE_GROUP, groupCreateHandler);
        handlerMap.put(Command.GROUP_MESSAGE, groupMessageHandler);
        handlerMap.put(Command.MESSAGE_REQUEST, messageHandler);
    }
    
	...do...{
	       handlerMap.get(msg.getCommond()).channelRead(ctx, msg);
	}    
     
```

但是调试的时候发现，map里的value都是空的。原因就是spring的注入是发生在构造函数之后的。我们可以将初始化代码放个一个单独的函数里并加上`@PostConstruct`注解。

如下

```java
  @PostConstruct
    private void init() {
        handlerMap = new HashMap<>();
        handlerMap.put(Command.CREATE_GROUP, groupCreateHandler);
        handlerMap.put(Command.GROUP_MESSAGE, groupMessageHandler);
        handlerMap.put(Command.MESSAGE_REQUEST, messageHandler);
    }
```







