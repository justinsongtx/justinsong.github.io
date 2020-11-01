---
title: '从编译失败了解Xcode中的各种架构 armv7,armv7s,arm64,i386,x86_64'
date: 2020-09-22 16:31:49
tags:
---

## 起因

![企业微信截图_8b7a02ae-f77c-459f-88a6-52b666835309](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/09/22/qi-ye-wei-xin-jie-tu8b7a02aef77c459f88a652b6668353.png)

更新了一下工程代码，发现编译失败了，我们的工程是子工程，主工程在link阶段找不到我们子工程的符号。

## 分析

![-w582](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/09/22/16007605923900.jpg)
细看错误提示写的是主工程在x86_64架构下找不到子工程的符号，好了，那就来看看子工程的工程设置

找到工程，打开build setting，真相了，支持的架构是armv7,arm64,i386，没有支持x86_64架构。
![-w550](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/09/22/16007606982543.jpg)

加上x86_64后编译成功
![-w253](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/09/22/16007608180330.jpg)

## 细品

那么这些armv7 arm64 i386 x86_64都是个啥？

* ARM架构

　　ARM架构过去称作进阶精简指令集机器（Advanced RISC Machine，更早称作：Acorn RISC Machine），是一个32位精简指令集（RISC）处理器架构，ARM处理器非常适用于移动通讯领域，符合其主要设计目标为低耗电的特性。

　　ARM和Intel处理器的第一个区别是，前者使用精简指令集（RISC），而后者使用复杂指令集（CISC)。
　　
* ARM处理器指令集

　　ARM指令集是指计算机ARM操作指令系统。

　　armv6、armv7、armv7s、arm64、arm64e都是arm处理器的指令集，所有指令集原则上都是向下兼容的。比如，你的设备是armv7s指令集，那么它也可以兼容运行比armv7s版本低的指令集：armv7、armv6。Xcode4.5起不再支持armv6。

　　苹果A7处理器支持两个不同的指令集：32位ARM指令集（armv6｜armv7｜armv7s）和64位ARM指令集（arm64）。i386｜x86_64 是Mac处理器的指令集。i386是针对intel通用微处理器32位处理器，x86_64是针对x86架构的64位处理器

　　i386通常被用来作为对Intel 32位微处理器的统称。X86-64可在同一时间内处理64位的整数运算，并兼容X86-32架构,x86_64是针对x86架构的64位处理器。当使用iOS模拟器的时候会遇到i386｜x86_64，iOS模拟器没有运行arm指令集，编译运行的是x86指令集，所以，只有在iOS设备上，才会执行设备对应的arm指令集。
　　

## iOS设备支持的指令集
* armv6:

　　iPhone, iPhone 3G, iPod 1G/2G

* armv7:

　　iPhone 3GS, iPhone 4, iPhone 4S, iPod 3G/4G/5G, iPad, iPad 2, iPad 3, iPad Mini

* armv7s：

　　 iPhone 5, iPhone 5c, iPad 4

* arm64:

　　iPhone X，iPhone 8(Plus)，iPhone 7(Plus)，iPhone 6(Plus)，iPhone 6s(Plus), iPhone 5s, iPad Air(2), Retina iPad Mini(2,3)

* arm64e:

　　iPhone XS\XR\XS Max
* i386：

    模拟器32位处理器测试需要架构，现在已经没有32位的模拟器，这个选项告别舞台了

* x86_64：

    模拟器64位处理器测试需要架构
    
　
## XCode设置

* Architectures

　　指定工程被编译成支持哪些指令集类型，而支持的指令集越多，就会编译出很多个指令集代码的数据包，对应生成二进制包就越大，也就是ipa包越大。

　　现在XCode->build setting 中Architectures的默认值是Standard architectures- $(ARCHS-STANDARD)， $(ARCHS-STANDARD)的值如下图所示：
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/09/22/16007612106947.jpg)

　　地址：https://pewpewthespells.com/blog/buildsettings.html#current_arch


* Valid Architectures
　　该编译项指定可能支持的指令集，该列表和Architectures列表的交集，将是Xcode最终生成二进制包所支持的指令集。

　　比如，你的Valid Architectures设置的支持arm指令集版本有：armv7/armv7s/arm64，对应的Architectures设置的支持arm指令集版本有：arm64，这时Xcode只会生成一个arm64指令集的二进制包。

　　减少安装包中的指令集数据包可以减小打包ipa的大小。

* Build Active Architecture Only:

　　指明是否只编译当前连接设备所支持的指令集。
　　
　　默认Debug的时候设置为YES，Release的时候设置为NO。设置为YES是只编译当前的architecture版本，生成的包只包含当前连接设备的指令集代码。设置为NO，则生成的包包含所有的指令集代码（上面的Valid Architectures跟Architectures的交集）。因此为了调试速度更快，则Debug应该设置为YES。

　　特殊：设置此值为YES，如果连接的设备是arm64的（ iPhone 5s，iPhone6（plus）等），则Valid Architecture 中必须包含arm64, 否则编译会报错.
　　
　　
## XCode中应该使用的配置

大多数工程的设置：arm64 armv7 x_86_64

剩余疑问：为什么不用指定armv7s和arm64e？ 

　
本文参考以下文档：
https://www.cnblogs.com/lulushen/p/8135269.html
https://www.jianshu.com/p/e01e3ee5a612