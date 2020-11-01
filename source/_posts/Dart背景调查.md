---
title: Dart 背景调查
date: 2020-06-29 01:53:54
tags:
---

# Dart出世

2011年10月10日，星期一，Goolgle发布了一篇名为《Dart: A language for structured web programming》（Dart：结构化的Web编程语言）的文章，公布Dart语言的出生。

# 设计初衷

摘一段原文：
> Dart’s design goals are:
> * Create a structured yet flexible language for web programming.
> 
> * Make Dart feel familiar and natural to programmers and thus easy to learn.
> 
> * Ensure that Dart delivers high performance on all modern web browsers and environments ranging from small handheld devices to server-side execution.

Dart的设计目标是：
* 创建用于Web编程的灵活结构化语言。
* 在使用Dart的时候，让开发者感到熟悉和顺手，从而易于学习。
* 确保Dart在所有移动设备、所有Web浏览器 以及服务器上提供高性能的服务。

看的出来, Dart的设计就是冲着多平台统一语言来的，在ios、安卓、web页面、后台的开发上都有做不错的支持，尤其是客户端的开发，原来两份代码的工作，Dart可以用一份代码搞定，开发成本大大节省。

# 低谷

Dart如今虽然风光无限，但在其实直到2018年Dart还是"最啃爹语言榜首"，而且就业机会极少，为什么？因为没有可心竞争力。

![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/06/29/15933679225129.jpg)

![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/06/29/15933681049121.jpg)


# 新的纪元

2018年12月4日，谷歌发布Flutter 1.0版本，其标题为《Flutter 1.0: Google’s Portable UI Toolkit》（Flutter 1.0:谷歌的可移植UI工具箱），Flutter从此问世。

Flutter的出现掀起来一波移动开发的热潮。使用它可以直接开发Android和iOS应用。其最大的特点就是一套代码多平台运行、高性能和Hot Reload（热重载）。谷歌即将发布Fuchsia系统就以Flutter为主要开发框架。Flutter采用Dart作为其底层语言。Dart也由于Flutter美好未来而得到众多开发者的青睐，从低谷一跃成为了2019年GIT上项目增长最多的语言。

并且Flutter和Dart都是开源项目，这为深入理解和调试敞开了大门。

![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/06/29/15933688221710.jpg)

# 教程

* Dart教程：

    * Dart语法教程：https://www.dartcn.com/guides/language/language-tour
    * 视频教程：https://www.youtube.com/watch?v=Ej_Pcr4uC2Q&t=1084s
    * 官方文档：https://dart.dev/guides
    * 静态检查工具：https://dart.dev/guides/language/analysis-options
    * 《Effective Dart》 https://dart.cn/guides/language/effective-dart
    * 《Linter for Dart》https://dart-lang.github.io/linter/lints/index.html
    * 《Dart Style Guide》http://dartdoc.takyam.com/articles/style-guide/

* Flutter教程

    https://www.youtube.com/watch?v=x0uinJvhNxI

# 最后

Dart的出世无疑是在全栈的高墙上搭了一把梯子，以后安卓 ios web 后台，4端的工作，只需要一个人就能完成了、，这可能会带来两个结果：1. 公司产能暴涨，有精力发展更多的产品。2. 公司的产能过剩，开发人力供大于求，裁员减招。希望我们都不会面临第二种情况。

