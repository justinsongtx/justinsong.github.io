---
title: 从头编译MNN iOS&MacOS静态库笔记
date: 2021-08-17 10:24:13
tags:
---

# 先下载代码
git clone老超时,直接下载源码包，这里使用的是MNN 1.1.1

# 开始编译iOS MNN

MNN编译步骤：
> 1. cd /path/to/MNN
> 2. ./schema/generate.sh
> 3. ./tools/script/get_model.sh（可选，模型仅demo工程需要）。注意get_model.sh需要事先编译好模型转换工具，参见这里。
> 4. 在macOS下，用Xcode打开project/ios/MNN.xcodeproj，点击编译即可
> 如果需要使用Metal后端，需要将mnn.metallib拷贝至应用的main bundle目录下，可以参考Playground应用Build Phases中的Run Script。

源文档：文档:https://www.yuque.com/mnn/cn/build_ios


下面是我在编译中遇到的一系列问题：

1. 按着文档一步步执行,到第3步命令执行失败:
    
    ```
    ➜  MNN-1.1.1 ./tools/script/get_model.sh
    can't find ../build/MNNConvert, building converter firstly 
    ```
原因是需要事先编译好模型转换工具, 编译转换工具的文档:https://www.yuque.com/mnn/cn/cvrt_linux ,照着文档一步步做,过程需要一段时间,,,,,,

1. 下载模型失败

    安装好模型转换工具,返回MNN更目录重新执行`./tools/script/get_model.sh`下载模型,如果不需要跑demo可以不下载,我为了方便验证MNN静态库的功能,所以要下载,过程又是一段漫长的等待,,,,,,
        
    然而网络不行,模型下载一直失败,看看执行的这个get_model.sh脚本里写的啥,把下载的链接抠出来放浏览器下载,发现也不行,尝试了n个代理节点之后,终于成功了. 如果你也卡在这部可以试试把模型链接放到浏览器打开试试,如果打不开,就别等了,浪费生命,其中一个模型链接:https://raw.githubusercontent.com/shicai/MobileNet-Caffe/master/mobilenet.caffemodel ,浏览器打开后下载不动,就是网络不行.
    
    为了让大家不在经历和我一样的痛苦过程,我把下载好的模型打包上传了,下载后替换MNN根目录中的`resource`文件夹即可.
        百度网盘: https://pan.baidu.com/s/1IHd0gS72vUa_glUzRcx6vg 
        提取码: b2jx

1. project编译成功但没效果
    
    下载模型完成,打开根目录下的project/ios/MNN.xcodeproj,编译运行成功,控制台输出均正常,但没有UI显示,和功能展示,心里有点不稳当,再打开根目录下demo/iOS,试试demo效果,需要先安装pod依赖,执行`pod install`,漫长的等待再次来临,,,,,
