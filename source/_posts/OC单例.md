---
title: OC单例
date: 2021-11-09 21:09:47
tags:
---



```

+ (instancetype)sharedInstance {
    static CameraView *single = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        single = [[self alloc] init];
    });
    return single;
}

+ (instancetype)allocWithZone:(struct _NSZone *)zone {
    static CameraView *singleClass = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        singleClass = [super allocWithZone:zone];//最先执行，只执行了一次
    });
    return singleClass;
}
```