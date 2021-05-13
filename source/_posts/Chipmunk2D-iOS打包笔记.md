---
title: Chipmunk2D iOS打包笔记
date: 2021-05-13 15:58:08
tags:
---

### 走的弯路

开始打开Chipmunk文件夹后，看到有CMakeList.txt，惯性的开始写脚本使用cmake打包，结果找不到opengl，绕了一大圈才发现根目录下有生成好的XCode工程😂。
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/05/13/16208929982115.jpg)

### 打包

打开根目录下的xcode/Chipmunk7.xcodeproj

* 选择scheme为Chipmunk-iOS
* 修改config为Release
* 在build setting中修改bitcode为YES
* 架构支持改为arm64真机和x86_64模拟器
* 打包、拿产物、合并多架构、完成。
