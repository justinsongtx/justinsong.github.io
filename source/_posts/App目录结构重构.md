---
title: App目录结构重构
date: 2020-09-21 00:39:10
tags:
---

> 日前与rucasli一起整理了兴趣部落的文件夹结构，重新区分出了App通用的组建，下面做个小结。

### 一。问题背景

目前兴趣部落目录结构凌乱，文件夹缺少归类，代码乱放情况较多，大大增加了代码维护成本。

* 公共组件问题

    * 目录凌乱
    
        随着代码的增长，文件目录开始凌乱，`公共组件`没有放到指定文件夹下，不便于后续`移植`。

    * 功能实现重复

        通过`review`代码发现，通用功能有不少`重复的实现`，其中较严重的甚至有7处重复实现，反思原因，`应该不知道这个功能已经实现or找不到该功能的实现文件`。本次重构后公共功能统一目录，并整理通用功能目录，可解决这个问题。
  
  * 缺少子分类

     反观以前的公共组件文件夹，大部分组件都直接`平铺`在一个文件夹下，缺少`竖直分类`，一眼看过去有些凌乱，不利于开发找代码。
	![](http://km.oa.com/files/photos/pictures/201705/1495799323_10_w496_h1204.png)

* 业务目录问题

    * 各业务缺少子分类
    
      各个业务文件夹下的代码，`缺少二级目录`，平铺情况较多，维护成本较高。
	![](http://km.oa.com/files/photos/pictures/201705/1495799362_18_w586_h1436.png)





### 二。解决方案

问题的原因：

    1. 老分类规则粒度较粗
    2. 开发中代码乱放
    
    
针对解决：

    1. 重新制定规则，细分文件夹种类
    2. 提高开发的归类意识&&编写codedog定时检查文件夹目录结构

    
    

重新制定的分类规则尽可能简单明了，降低学习成本和开发成本，让开发同事一秒找所属的文件夹。


* 大原则
    
    跨App通用的文件放`TribeMartrix`文件夹，不通用的放`Tribe`文件夹。
	![](http://km.oa.com/files/photos/pictures/201705/1495799431_75_w1568_h712.png)






* 工程二级文件夹原则（TribeMartrix和Tribe文件夹共用这套规则）
    
    * `AppDelegate`，放AppDelegate的子类及其Categories
    * `Categories`，放各种公用分类（eg：NSString + XXX， NSData + XXX）
    * `Public`，放公共功能or定义
    * `ThirdParty`，放第三方库or源码
    * `Biz`, 放业务代码
    
	![](http://km.oa.com/files/photos/pictures/201705/1495799473_76_w1554_h1288.png)


* Public文件夹细分
    * `Cache`, 缓存代码
    * `Controller`，公用UIController子类
    * `DataBase`，数据库代码
    * `Logic`，公用逻辑代码
    * `Macros`，公用宏定义
    * `Model`，公用数据模型
    * `NetWork`，网络层代码
    * `Service`，服务代码
    * `Utils`，公用小功能代码
    * `View`，可复用的View或UI控件
    
	![](http://km.oa.com/files/photos/pictures/201705/1495799506_13_w588_h1362.png)


* `Biz`, 放业务代码
    * `Model`，业务Model 
    * `View`，业务View
    * `Controller`，业务Controller
    * `Manager`，业务Manager，非必选，如果有请归类成文件夹
    * `Logic`，逻辑代码，独立出来可以为Controller肩负，如果有请归类成文件夹
    * `其他`，如果还有其他分类，各业务自行归类
    
	![](http://km.oa.com/files/photos/pictures/201705/1495799538_47_w572_h734.png)

        

### 三。App级通用组建功能梳理计划

计划梳理TribeMartrix（跨App通用）的功能，在公共方法中添加注释，然后用<font color=red>AppleDoc</font>生成文档，由于通用功能较多，后续这里会持续更新文档。
(wiki:http://tapd.oa.com/MonaLisa/markdown_wikis/view/#1010101141006004831)

### 四。工作排期

人力：justinsong，rucasli
文件夹改动容易引起树冲突，安排在版本发布间隔的时候来整理可以规避冲突，下个版本发布间隔再抽时间来继续后面3项工作。

- [x] 新目录结构搭建--0.5day（done）
- [x] 已有代码文件整理到新目录中，抽离App通用功能--6day（done）
- [x] 重复功能剔除--0.5day（done）
- [ ] 已有通用功能文档持续梳理--工作量较大，待持续梳理
- [ ] App框架能力抽离--6day
- [ ] 文件夹codedog程序编写&&推行--2day

### 五。总结

这次和rucas一起梳理文件夹和代码，在梳理中发现很多文件的物理目录和工程目录不对应，还有很多工程中没有用到的废弃文件还存放在物理目录下，梳理中已经全部删除，后续开发中希望大家注意物理目录和工程目录的对应，遵循文件夹分类规则，大家一起努力维持兴趣部落代码结构的纯净 ：）

（完）

