---
title: blender 如何打印选中点的索引
date: 2021-06-15 11:44:41
tags:
---

## STEP1:
编辑模式下选中点

## STEP2：
切换物体模式（object model），不切换打印不出来，二逼blender

## STEP3：
工具栏选中python控制台，输入命令：

```
Verts = [i.index for i in bpy.context.active_object.data.vertices if i.select]

print(Verts)
```

（完）