---
title: XCode中查看float数组
date: 2021-06-17 11:39:38
tags:
---

# 病情
XCode查看float指针时，只能看到第一个元素的值，无法查看后续的值.
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/06/17/16239022086217.jpg)

# 治疗

在参数查看器中输入以下代码，其中nativeString是float指针的地址，将他强转成结构体查看。
```
typedef struct {         float a[1600];     } X;     X *x = (X*)nativeString; x;
```

# 疗效
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/06/17/16239026141773.jpg)
