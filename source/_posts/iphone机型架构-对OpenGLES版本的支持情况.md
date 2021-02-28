---
title: iphone机型架构 & 对OpenGLES版本的支持情况
date: 2021-02-28 18:04:08
tags:
---

## 架构机型对照

**armv7**：iPhone4｜iPhone4S｜iPad｜iPad2｜iPad3(The New iPad)｜iPad mini｜iPod Touch 3G｜iPod Touch4

**armv7s**：iPhone5｜iPhone5C｜iPad4(iPad with Retina Display)

**arm64**： iphone8 | iphone8 plus | iphoneX | iphone7 | iphone7 plus | iPhone6s | iphone6s plus｜iPhone6｜ iPhone6 plus｜iPhone5S

**arm64e**：iPhone XS | iPhone XS Max | iPhone XR | iPhone 11 | iPhone 11 Pro | iphone 11 Pro Max

**i386**是针对intel通用微处理器32位处理器

**x86_64**是针对x86架构的64位处理器


## OpenGLES支持情况

1. **IOS从IOS7以上版本开始支持ES 3.0**

2. **iphone 5及以下的机器只支持OpenGL2，不支持OpenGL3（包括iphone5c）**

3. **iphone 5s及以上的机器开始支持OpenGL3.0**

参考文档：https://www.cnblogs.com/singmelody/p/3774638.html
https://zhidao.baidu.com/question/1833302402268128980.html
