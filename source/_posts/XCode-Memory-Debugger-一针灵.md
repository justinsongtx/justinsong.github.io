---
title: XCode Memory Debugger 一针灵
date: 2023-01-30 11:08:14
tags:
---


## 背景
iOS 内存分析除了使用 Instrument 以外，Xcode 中还有 Memory Debugger。Memory Debugger有可视化界面，有内存申请的树状图，也可以将抓取的数据导出成.MEMGRAPH文件，再借由命令行分析。

## 如何开启
先打开scheme中的Malloc Stack, 否则看不到backtrace（内存申请的堆栈）
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750481365786.jpg)

运行app，等待内存稳定后，点击
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750481447064.jpg)

or 点击，选择 `View Memory Graph Hierarchy`
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750481573665.jpg)
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750481663251.jpg)


## 交互解释

* 概览
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750481829575.jpg)

* 查看泄漏对象 & 剔除系统内容
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750481904907.jpg)


## 导出.MEMGRAPH
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750482005712.jpg)

接下来就可以使用命令行来分析 .MEMGRAPH 文件


## 命令行 分析.MEMGRAPH 


#### vmmap

vmmap 通过将进程所占虚拟内存信息打印出来，以提供一些关于进程内存消耗更高级别的分析。

* 命令
    `vmmap App.memgraph` （全量信息）
    `vmmap —summary App.memgraph` (摘要)

* `vmmap App.memgraph`

能查看所有内存区域的具体信息 首先是 non-writeable region，比如可执行文件、程序文本等：
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750482133755.jpg)
接着是 writeable region，这里就是 app 进程的堆所在的位置：
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750482212253.jpg)

最后是一些摘要信息和不同类型的虚拟内存所占不同内存区块的大小，如果我们关注内存问题，应该关注 DIRTY SIZE 和 SWAPPED SIZE 这两列，分别表示脏内存大小和交换内存大小
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750482297774.jpg)


#### leaks

* 命令
    `leaks App.memgraph` （查看泄漏对象）
    
* `leaks App.memgraph`
    用 leaks 命令，在命令行中也能看到对应的信息，而且不仅是泄漏的对象，还有造成泄漏的回溯堆栈(前提是启用了 Malloc Stack)。
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750482421512.jpg)


#### heap

有时候我们使用 vvmap 后发现堆内存很大，但是仅此而已，不会有跟多的信息 这时候可以借助 heap 命令，heap 可以帮助显示堆中的内存分配，用于追踪非常复杂的分配。

* 命令
    `heap App.memgraph`
    `heap —sortBySize App.memgraph`
    `heap App.memgraph -addresses all | <classes-pattern>`
    
* `heap App.memgraph`
    展示类名、这类对象的数量、平均大小、总大小 看这张图可以发现，heap 默认是按数量来排序的，但是按此排序数量最多的大多是是系统对象，这样我们其实看不出问题在哪里.
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750482536716.jpg)


* `heap —-sortBySize App.memgraph` or `heap -s App.memgraph`
    我们可以传递参数，让 heap 按照 size 来排序： heap —sortBySize App.memgraph
    ![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750482627436.jpg)


* `heap -addresses (all | <classes-pattern>) App.memgraph`

    上图可以看到 non-object 占了非常多内存，但这还不够，还需要知道这些对象是怎么形成的
    `heap -addresses (all | <classes-pattern>) App.memgraph`
    当通过 -addresses 将具体的类名传递给 heap 指令时，heap 会罗列出所有指定类的每个实例的地址，有了这些地址，我们就能进一步研究这些对象是哪里来的了
    ![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750482800443.jpg)


#### malloc_history

显示具体 .memgraph 文件的具体对象地址的堆栈 我们将上面 NSConcreteData 的其中一个地址输入，得到回溯记录，并且在堆栈中找到了和 App 相关的方法

* 命令
    `malloc_history -callTree <memgraph> <address>`
    
* `malloc_history -callTree <memgraph> <address>`
    显示具体 .memgraph 文件的具体对象地址的堆栈 我们将上面 NSConcreteData 的其中一个地址输入，得到回溯记录，并且在堆栈中找到了和 App 相关的方法
    ![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750482901524.jpg)
    non-object 类型的对象抓不到，可能超出Memory Debugger的可以抓的范围了。第二名是uint指针也搜uint指针地址太多了，不适合用命令行分析，改用CFString尝试。
    ![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2023/01/30/16750483163894.jpg)

如上图，可以看到该对象的申请堆栈。

综上，我们就通过命令行追踪到了大内存消耗的具体原因

## 总结
上面提到的几种指令，各有不同的作用： malloc_history：关注对象的创建 Leaks：关注对象引用，内存泄漏 vmmap、heap：关注一个属于或者一个实例有多大


## 参考：
https://developer.apple.com/videos/play/wwdc2018/416/

https://linkexin.github.io/notes/Xcode-Memory-Debugger

https://juejin.cn/post/7107507601545363493