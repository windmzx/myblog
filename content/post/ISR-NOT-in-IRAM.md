---
title: ISR NOT in IRAM
date: 2019-08-30 21:22:15
tags:
---


最近买了几个ESP8266模块，这应该是市面上最便宜的支持wifi的芯片了，而且本身可以作为MCU来使用。我用其试验了下智能家居，完整实现了自动配网，控制led，连接到百度的物联网平台，并使用mqtt和微信小程序进行通信。发现其实卖智能设备还是很挣钱的，相对于传统的设备，智能设备在硬件上的成本增加的是非常少的，重难点还是在构建生态的闭环上。


代码预计会贴上来。其中实现一个长按按钮重值设备的时候遇到了一点问题，发现网上的资料还是比较少的，记录在此。

使用中断对按键进行检测的时候，将代码编译好之后，设备在启动的过程中就会重启，并打印异常栈。其中有错误信息如下ISR NOT in IRAM，去群里请教大佬，都说没有遇到这种情况。后在github上找到了答案。
[github](https://github.com/esp8266/Arduino/issues/6087#issuecomment-492015020)
是因为sdk更新，对安全进行了加强，因为我们需要对中断函数进行改进，添加宏定义`ICACHE_RAM_ATTR`
```
void ICACHE_RAM_ATTR  interFunction();
```
就可以解决问题。