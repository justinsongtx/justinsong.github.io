---
title: iOS库中bitcode、架构、静态动态的判断
date: 2021-05-11 15:00:11
tags:
---

### 判断是否支持bitcode
执行命令：
```
otool -arch arm64 -l xxxxx.a(二进制文件) | grep __bitcode | wc -l
```

* 结果非0： 支持bitcode
* 结果0：不支持bitcode

### 判断库中的架构

执行命令：
```
lipo -i xxx.a(二进制文件)
```
支持的架构会打印出来

### 判断静态库还是动态库

执行命令：

```
file xxx.a(二进制文件)
```
* 静态库包含`current ar archive random library`字样。
* 动态库包含`dynamically linked shared library`字样

