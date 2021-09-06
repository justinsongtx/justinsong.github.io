---
title: blender 如何打印选中点的索引
date: 2021-06-15 11:44:41
tags:
---

# 打印选中点的索引

### STEP1:
编辑模式下选中点

### STEP2：
切换物体模式（object model），不切换打印不出来，二逼blender

### STEP3：
工具栏选中python控制台，输入命令：

```
Verts = [i.index for i in bpy.context.active_object.data.vertices if i.select]

print(Verts)
```

# 已有索引，通过控制台选中这些点


```
keyindex = [0, 1, 2, 3]
for i in bpy.context.active_object.data.vertices: 
    if i.index in keyindex:
        i.select = True
```
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/09/06/16309172817051.jpg)

（完）