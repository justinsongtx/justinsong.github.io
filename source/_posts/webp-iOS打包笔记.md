---
title: webp iOS打包笔记
date: 2021-05-13 16:50:47
tags:
---

### 修改原有打包脚本

原本的打包脚本中已经支持了bitcode，还需要修改的是把framework包装改成.a。

webp工程在根目录下自带了iosbuild.sh打包脚本，表扬！基于iosbuild.sh直接修改就好了，修改这么几处：

##### framework改为.a包装
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/05/13/16208961142445.jpg)

##### 干掉不用的架构
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/05/13/16208961375689.jpg)

##### 修改产物后缀
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/05/13/16208961590645.jpg)

然后执行脚本，拿产物，收工。