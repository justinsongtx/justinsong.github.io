---
title: 解决SourceTree反复要求输入密码
date: 2021-06-14 16:15:13
tags:
---

# 病情
sourcetree提交的时候不停地弹出窗口，要求输入密码，输入之后并没有卵用，逼死强迫症了！！！

![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/06/14/16236585439228.jpg)


# 治疗

控制台输入命令:

```
git config --global credential.helper osxkeychain
```

打开sourcetree，这时会要求输入一遍开机密码，记得点击始终允许，完事！出院！