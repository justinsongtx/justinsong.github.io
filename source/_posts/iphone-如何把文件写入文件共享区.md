---
title: iphone 如何把文件写入文件共享区
date: 2021-08-27 17:23:33
tags:
---

# 背景
保存的文件想要像这样，能够在mac上访问（mac上拖拽文件到finder中可以拷贝，ctrl c + ctrl v不行）
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/08/27/16300563829976.jpg)

# 怎么做

**STEP1:**

代码中保存文件到NSDocumentDirectory文件夹:


```
 
NSString *documentPath = [NSHomeDirectory() stringByAppendingPathComponent:@"Documents"];
int index = [ViewController index];
__block NSString *filePath = [documentPath stringByAppendingFormat:@"/%d.png", index];
    
```

**STEP2:**

配置info.plist文件

Supports opening documents in place = YES

Application supports iTunes file sharing  =  YES
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/08/27/16300565284482.jpg)

(完成)

