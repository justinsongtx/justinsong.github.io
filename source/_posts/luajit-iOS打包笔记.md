---
title: luajit iOS打包笔记
date: 2021-05-13 15:56:12
tags:
---
# luajit iOS打包笔记

### 查看文档
在doc/intall.html中提示，有关于iOS打包的指引：
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/05/13/16208916328164.jpg)

使用这个作为打包命令，打包出产物后，产物**不支持bitcode**。

### 添加bitcode支持

转而在工程中搜索bitcode关键字，企图找到bitcode的开关，结果没有。又在转而想要自己添加bitcode的编译选项，[参考文档](https://www.geek-share.com/detail/2789992000.html).文档中提示想要支持bitcode，需要在CFlags中添加**-fembed-bitcode**和**-fembed-bitcode-maker**。

知道了要添加的选项，接下来就是找到要添加的地方，打开src/MakeFile，在MakeFile中搜索CFLAGS，并没有使用，CFLAGS是系统变量他没用到我直接添加也是生效的,于是增加bitcode选项：
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/05/13/16208924923660.jpg)

重新打包，解决！