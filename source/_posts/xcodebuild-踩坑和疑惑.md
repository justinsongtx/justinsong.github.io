---
title: xcodebuild 踩坑和疑惑
date: 2021-05-12 14:43:24
tags:
---

# 不同打包选项的产物目录

##### xcodebuild -target

```
xcodebuild archive -project ../ios/kiwi/kiwi.xcodeproj -target kiwi -configuration Release -arch "x86_64" -sdk $XCODE_SIMULATOR_SDK
```
打包时如果用**-target**选项，那么产物会打包到当前工程目录的“build/UninstalledProducts/iphoneos/”下面。

##### xcodebuild -scheme


```
xcodebuild archive -project ../ios/kiwi/kiwi.xcodeproj -scheme kiwi -configuration Release -arch "x86_64" -sdk $XCODE_SIMULATOR_SDK
```
打包时如果用**-scheme**选项，那么产物会打包到DerivedData文件夹下面，例如:
```
/Users/jenkins/Library/Developer/Xcode/DerivedData/kiwi-haukwgjiorhjwxbmduttecrfbfml/Build/Intermediates.noindex/ArchiveIntermediates/kiwi/IntermediateBuildFilesPath/UninstalledProducts/iphoneos/
```

##### 修改产物目录的疑问

修改DerivedData目录后，无论是在project  setting还是xcode setting中修改，用beyondcompare对比修改前后的工程文件，均没有变化。

这就导致无法提交变更，不能同步给别人，目前还不知道这个修改到底改的哪个文件，很疑惑。

# 采坑

工程结构如下：
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/05/12/16208027369646.jpg)

##### 背景

kiwi和kiwi-cocos是两个静态库，kiwi会依赖的kiwi-cocos。

##### 问题

使用xcodebuild -target打包kiwi静态库时，总是提示找不到kiwi-cocos的库，但实际libkiwi-cocos的产物已经打包出来了。

##### 尝试

过程中尝试了改用**xcodebuild -scheme**打包，可以成功打包，但产物目录在DerivedData路径下：
```
/Users/jenkins/Library/Developer/Xcode/DerivedData/kiwi-haukwgjiorhjwxbmduttecrfbfml/Build/Intermediates.noindex/ArchiveIntermediates/kiwi/IntermediateBuildFilesPath/UninstalledProducts/iphoneos/
```
其中“kiwi-haukwgjiorhjwxbmduttecrfbfml”路径是乱码，没有找到获取这个路径的办法，产物虽然打出来了，但脚本中拼不出产物路径，搜索后发现，网上的静态库打包脚本都是使用**xcodebuild -target**的方式，然指定工程的相对路径，找到产物，于是再次改回**-target**。

##### 最终解决

macos的工程依赖和ios的结构相同，mac使用的也是**xcodebuild -target**，却可以打包成功，这让我有点晕。

最后抱着试一试的心态删除重新添加了ios工程中对kiwi-cocos的依赖，妈的解决了！一脸懵逼！
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/05/12/16208034012530.jpg)


虽然解决了，但并不清楚原因，推测是依赖的kiwi-cocos产物md5有变化。这个问题暴露了我对工程配置的理解不足，后续要补齐对xcode工程设置和打包过程的短板。

