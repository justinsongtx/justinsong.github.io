---
title: NSString类文件夹路径的处理方法
date: 2020-12-31 10:53:24
tags:
---

## NSString类路径的处理方法:

### 文件路径的处理:

NSString *path = @"/Uesrs/apple/testfile.txt"

常用方法如下:

1. 获得组成此路径的各个组成部分，结果：（"/","User","apple","testfile.txt"）

    -(NSArray *)pathComponents;

2. 提取路径的最后一个组成部分，结果：testfile.txt

    - (NSString *)lastPathComponent;

3. 删除路径的最后一个组成部分，结果：/Users/apple

    - (NSString *)stringByDeletingLastPathCpmponent;

4. 添加到路径的末尾，结果：/Users/apple/testfile.txt/app.txt

    - (NSString *)stringByAppendingPathConmponent:(NSString *)str;

5. 删除路径最后部分的扩展名，结果：/Users/apple/testfile

    - (NSString *)stringByDeletingPathExtension;

6. 路径最后部分追加扩展名，结果：/User/apple/testfile.txt.jpg

    - (NSString *)stringByAppendingPathExtension:(NSString *)str;

转自：https://www.jianshu.com/p/16e948b3d1d2