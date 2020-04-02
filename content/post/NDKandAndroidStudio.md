---
title: 在AndroidStudio里进行NDK开发时mips支持不可用的问题
date: 2019-03-27 20:21:39
tags:
---

我们在开发安卓应用的时候,有的时候需要调用opcv等C++代码.最近我在AS中配置ns的时候遇到了一个问题如下
`No toolchains found in the NDK toolchains folder for ABI with prefix: mips64el-linux-android`
从字面意思来说,缺少了mips平台的交叉编译工具链,但是我们ndk都是as自动下载,不应该出现这么低级的错误.google一下解决方案，教程是让你去[谷歌的开发者网站](https://developer.android.com/ndk/downloads/?hl=zh-en)手动下载一个ndk的包，然后把缺少的文件拷贝进去，但是我们从网站上下载完之后，打开压缩包,发现里面还是没有这个文件夹。

在google的[github](https://github.com/google/filament/issues/15)一个项目里,发现了一些人在讨论这个问题. 在某个ndk版本以后，mips已经从支持里remove掉了。因此在官网的下载的最新sdk也是没有mips支持的。 但是由于demo里的gradle可能是这个时间点之前的版本。因此在gradle编译的时候去ndk里搜索，发现没有mips的支持就会报错，因为解决方案就是更新gradle版本。这时候你就会发现会提示找不到gradle版本，或者对gradle plugin报错。因为gradle版本和gradle plugin版本病不是一个，但是两者存在对应关系。如果不匹配仍然还会报错. 在谷歌的[开发者网站](https://developer.android.com/studio/releases/gradle-plugin.html#updating-gradle)找到了这个页面上面写了gradle的版本 和plugin的版本对于关系,我们将gradle配置里修改成一样就可以了.

