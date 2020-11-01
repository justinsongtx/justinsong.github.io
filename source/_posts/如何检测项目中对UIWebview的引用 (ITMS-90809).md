---
title: 如何检测项目中对UIWebview的引用 (ITMS-90809)
---


自从去年开始苹果就发布了通知，禁用UIWebview，替换为WKWebview，继续使用会在提交时收到苹果粑粑的警告:

> ITMS-90809: Deprecated API Usage - Apple will no longer accept submissions of new apps that use UIWebView as of April 30, 2020 and app updates that use UIWebView as of December 2020. Instead, use WKWebView for improved security and reliability. Learn more (https://developer.apple.com/documentation/uikit/uiwebview).

UIWebview可能存在与业务代码中、pod中、动静态库中，单单使用Xcode的查找是不行的。

那么，要把大象装冰箱，拢共分几步？ 答：三步

### 第一步
控制台打开工程目录
    
### 第二部
输入命令
```
grep -R UIWebView *
```
### 第三步：
把搜索出来的结果挨个干掉or升级，AFNetwork升级最新，微信SDK升级1.8.2以上，等等，注意有用到UIWebviewDelegate的也要干掉，不要给自己找麻烦。
    

完成，鞠躬下台。
    