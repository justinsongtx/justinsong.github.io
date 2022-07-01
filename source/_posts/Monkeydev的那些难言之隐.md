---
title: Monkeydev的那些难言之隐
date: 2022-03-26 22:09:42
tags:
---

随着MacOS更新，XCode更新，MonkeyDev开始有各种编译问题，安装问题，开一个文章记录一下解决办法。

### 难言之隐——尿频

* 病情
    编译报错，找不到libstdc++库
    ```
    file not found: /usr/lib/libstdc++.dylib
    ```
* 分析
    因为XCode10之后删除了libstdc++库，而monkeydev已经很久没维护了，这货不支持新的c++库。
    
* 治疗
    下载libstdc++库，放到XCode里
    https://github.com/devdawei/libstdc-

### 难言之隐——尿急

* 病情
    运行crash，在fishhook.c里面
    
* 分析
    看不出来原因，网上一搜，解决了
    
* 治疗
    下载最新的fishhook，替换工程中的文件
    https://github.com/facebook/fishhook

### 难言之隐——尿不尽

* 病情
    
    打不出可执行程序，clean重编，手机电脑重启，后又正常
    ![w200](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2022/03/26/16483050564373.jpg)    
    
    安装失败，clean重编，手机电脑重启，后又正常
    ![w200](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2022/03/26/16483051434349.jpg)

* 分析
    先开始怀疑是升级XCode问题，老的逆向工程重复编译运行正常，未见明显异常。
    又怀疑砸壳没砸干净，于是把本地编译的ipa丢进monkeydev，发现也这样，本地编译的ipa没有壳，这个原因也排除了。
    不是工具原因，不是ipa原因，那就是工程设置问题了，这可难受了，变成力气活了。用beyondcompare对比两个工程，一个能反复运行，一个不行，一项项对比。发现新创建的工程info.plist的key变了，而且新建的Monkeydev工程里info.plist是空的（如下图），隐隐觉得就是这个原因，填上info.plist路径（“你的app名字/Info.plist”）, 果然正常了。
    ![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2022/03/26/16483057350303.jpg)

* 治疗
    填好info.plist路径，Build Setting->Info.plist File，填上，出院！
    ![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2022/03/26/16483059080441.jpg)

### 难言之隐——前列腺肥大

* 病情
    'Cycript/Cycript.h' file not found  
    
    ![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2022/07/01/16566665016623.jpg)

* 分析
    找不到Cycript.framework, 这玩意在theos里面，妈的，不知道为什么找不到了，也没链接这个framework。

* 治疗
    找到Cycript.framework，一般在"~/Users/zego/opt/theos/vendor/lib"里面, 把这个东西复制到工程里面或者"~/Users/zego/opt/MonkeyDev/Frameworks/"下面。
    ![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2022/07/01/16566667483184.jpg)

    把Cycript.framework拖进工程，链接到app target上面，注意链接顺序要在lib之前。
    ![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2022/07/01/16566674108754.jpg)

    编译-》运行-》出院！

（疗程结束）