---
title: MediaPipe iOS真机运行指引
date: 2020-11-08 22:44:51
tags:
---

### 使用Tulsi导出XCode工程

这步比较简单，按照官网即可，https://google.github.io/mediapipe/getting_started/building_examples.html#ios

### 如何设置证书在真机上运行

* STEP 1

删除XCode编译产物，保证工程干净 XCode菜单 Product -> Clean
 
* STEP 2

删除bazel编译产物，同样保证工程干净，在mediapipe文件夹中执行：
```
bazel clean --expunge
```   

* STEP 3
    
使用控制台下载编译时需要下载的文件，测试网络是否通畅，随便找个临时文件夹就好，在控制台中打开文件夹，然后执行：

```
https://github.com/bazelbuild/rules_cc/archive/master.zip
```

这个文件不需要翻墙就能下载，如果下载失败，请检查自己的网络设置，把代理该干掉的干掉。

* STEP 4

在mediapipe项目文件夹中执行：

```
python3 mediapipe/examples/ios/link_local_profiles.py
```
这条命令有两个功能，先说第一个功能：

1. 生成bundleID前缀
    生成的前缀会写到mediapipe/examples/ios/bundle_id.bzl中，长这个样子：
![-w636](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/11/08/16048448781636.jpg)
嫌它丑，也可以改成自己习惯用的，比如我就改成了"justinsong.Demo",这个前缀只生成一次，后面不会重复生成。

* STEP 5
    上一步第一次执行link_local_profiles.py脚本之后，生成了bundleID前缀，这时我们需要打开工程，为我们想要运行的例子修改bundleID，修改成 bundleID前缀+例子名的格式，修改后回车。
    
>     例如：比如前缀是09a8dab3-7aa0-419e-9e6f-2fa48b7495f8.mediapipe.examples，要运行的例子是FaceDetectionCpu，那么你target的bundleID就要写成"09a8dab3-7aa0-419e-9e6f-2fa48b7495f8.mediapipe.examples.FaceDetectionCpu"
    ![-w506](media/16037660886171/16048457811108.jpg)
    

* STEP 6
        
在mediapipe项目文件夹中再次执行：

```
python3 mediapipe/examples/ios/link_local_profiles.py
```

用到这个脚本的第二个功能：

1. 创建mobileprovision替身，放到各个example的文件夹中去
    这一步会根据bundle_id.bzl中的bundleID前缀匹配example文件夹中各个target的bundelID，如果前缀相同，脚本就在这个例子的文件夹中创建一个mobileprovision的替身。
    
>     注意！ 这一步想生效有个前提，是你的profile要在mediapipe/provisioning_profile.mobileprovision这个位置，否则失败，如果你的这个路径下没有mobileprovision，解决办法如下：
>     * 如果你已经有mobileprovision，直接复制到mediapile/midiapile/路径下，你可能有的这个文件在~/Library/MobileDevice/Provisioning\ Profiles/路径下面，拷一个没过期的过来，改个名字就行了
>     
>     * 如果你没有这个东西，那就要按照苹果官方指引生成下载再拷贝改名了

执行成功后，你的文件夹中会多一个mobileprovision的替身文件，且能看到原身的信息（如果看不到替身或点击替身找不到原身，回到STEP4重新做。）这时距离成功就差一步了 ：）
![-w761](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/11/08/16048463886571.jpg)

* STEP 7
    回到XCode工程中，选择想要运行的target，运行！
    
（结束）