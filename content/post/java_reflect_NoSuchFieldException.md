---
title: "Scala的case class无法进行反射的问题"
date: 2020-07-26T08:17:38+08:00
---

最近在研究Flink的广播流下发规则，动态的对数据流进行分组。在流中接受到了字符串的规则之后，我们解析这些字段，并且用反射的方法将字段值抽取作为key，让keyby算子可以进行分组操作。

由于代码是java+scala混合编写的。pojo是scala的

```scala
case class LoginEvent(userId: Long, ip: String, eventType: String, eventTime: Long)
```



使用反射之后发现提示找不到字段`NoSuchFieldException: userId`，对于scala了解不深，以为是scala代码编译之后和java编译之后，字节码不一样。那么首先回到我们熟悉的Java领域。

将其改成java的pojo

```java
public class LoginEvent {
    private Long userId;
    private String ip;
    private String eventType;
    private Long eventTime;

    public LoginEvent(Long userId, String ip, String eventType, Long eventTime) {
        this.userId = userId;
        this.ip = ip;
        this.eventType = eventType;
        this.eventTime = eventTime;
    }
    ...
}    
```

运行还是报错。搜索java+发射+`NoSuchFieldException`可以很容易搜索到，对于private修饰的变量，我们要使用`getDeclaredField`这个方法，直接使用`getField`是无法获得值的。

尝试修改还是报错，这回事权限不对，告诉我们priivate属性无法访问，直接调用`setAccessible`允许访问就可以了。

那么问题回来了，scala代码是不是也是因为同样的原因呢？

![](img.0xaa.top/20200726085911.png)

我们使用javap 反编译一下注意带-p指令，可以看到字段是final并且是ACC_PRIVATE的。case class帮们做了很多。因此这个就是case class不能使用`getField`进行反射的原因。
