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


# 模型旋转、平移、缩放

blender的场景是以三维坐标(0,0,0)为场景中心的。
shift+s选择“游标”--“中心点”，这就是我们的场景中心。
我们可以选择底部的“移动选择工具”，对模型操作，然后就可以在左边的“移动”下面输入xyz的相对移动数值，缩放、旋转也是一样。
我们还可以“N”键掉出右边的面板，在其中可以查看与设置模型的精确位移旋转缩放。
其实还有一种很方便的方法，选择模型，我们按“G”键接着直接按“X”直接填写要移动X的数值，在底部会直接显示数值。
按“S”按键，可以直接输入缩放的倍数数值，也可以直接再按X或Y或Z对模型进行准确倍数缩放，同理旋转就是R然后输入XYZ数值。
即使在“编辑模式”，我们也可以直接使用“G” 、“S”、“R”、然后对点线面进行精确的设置。

（完）