---
title: 如何解决LLDebugTool 不显示悬浮窗
date: 2022-02-01 17:45:40
tags:
---

* 删除SceneDelegate代理文件 (可选)
* 删除 Info.plist里面的Application Scene Manifest配置（一定要删除）
* 删除 AppDelegate代理的两个方法：
application:configurationForConnectingSceneSession:options:
application: didDiscardSceneSessions:
这两个方法一定要删除，否则使用纯代码创建的Window和导航控制器UINavigationController不会生效。


原文链接：https://blog.csdn.net/holdsky/article/details/102602213