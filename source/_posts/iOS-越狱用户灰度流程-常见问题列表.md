---
title: iOS 越狱用户灰度流程&&常见问题列表
date: 2020-09-21 00:40:56
tags:
---


一. 越狱用户灰度流程

     0.写在前面：

          使用plist安装，一般是企业级开发者账号不需要登录到APP STORE的IOS设备应用发布时所用到的技巧。

准备:
ipa和plist文件
一个HTML网页文件(告知iphone如何找到itms-services，已附上)
一个HTTP服务器(存放APP的服务器，就是提供ipa流量的服务器)
一个支持https的服务器，用于推送plist，本文上传到pub.idqqimg.com

备选:
一张二维码
 
PS:
从IOS7.1开始，http推送plist已经不好使，只能使用https推送
 
     1.rdm编包

          上架appstore之前，发布证书打包的ipa包只有越狱手机可以安装，灰度越狱用户时使用发布证书很合适。多得不说，工程中使用发布证书上库编包，准备好ipa和plist。（如果没有发布证书，可以找ladyli申请）。

     2.上传文件到服务器
     
             step1: 上传ipa到服务器，拿到ipa的url。
            服务器：http://turtle.oa.com/，注意存储类型选择CDN



             step2: 修改plist中的ipa地址，将上一步的url填到这里即可，并确认plist中的bundle id和发布证书的bundle id一致。
          
              step3:上传plist到https服务器，注意这里苹果要求必须是https，拿到plist的url。
              step4:拼接url,使用itms-services服务，
               URL：itms-services://?action=download-manifest&url=你的plist文件url，必须是https的哦
               例如：itms-services://?action=download-manifest&url=https://pub.idqqimg.com/pc/misc/files/20151027/8260f5e0328d48a7928e1ac55e682ee1.plist

          到这里为止已经可以体验安装流程了，将url复制到浏览器中，弹出下图提示框，点击安装可以正常下载安装，恭喜你！距离成功不远了！如果没有弹出提示或提示错误，请阅读第五条换证书注意事项和常见问题。
          

     3.开发html
         
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>应用名字</title>
</head>
<body>
<h1 style="font-size:80pt">如果点击无法下载安装，请复制超链接到浏览器中打开<h1/>
<h1 style="font-size:100pt">
<a title="iPhone" href="itms-services://?action=download-manifest&url=https://pub.idqqimg.com/pc/misc/files/20151027/8260f5e0328d48a7928e1ac55e682ee1.plist">
Iphone Download</a><h1/>
</body>
</html>

         使用拼好的URL 替换以上URL即可，保存，发布网页，完成。
          上一个例子：http://connect.qq.com/files/app.html，点击html中的链接即提示安装

     4.换证书注意事项

          a.工程中的sdk如果有依赖证书的bundle id的，sdk需要找提供者更新bundle id
          b.如果app中有push功能，客户端更换证书前要确认证书是否带有push功能，如果没有可以找rdm证书的同事添加，另外客户端更换了证书，wns也需要同样更换push证书（wns接口人： vanqfjiang(江且凡)），push证书制作见：http://km.oa.com/group/21900/articles/show/186606
          c.如果有接入日志上报和crash上报，上报平台的bundle id要使用证书的bundle id。
          d.rdm编包时如果勾选了“使用企业签名”，如下图，则编出来的结果会有几个，注意第一个名称中不带sign的ipa包，才是使用工程中的证书编出来的，第二个带sign的ipa是使用企业证书重签名打包的ipa，楼主在这卡了很久才发现这个问题。
          



二. 常见问题

在 iOS 9 中启动应用时，出现提示“未受信任的企业级开发者”
这样问题是因为在 iOS 9 以后的版本中，苹果对企业签名的应用做了更严格了限制。具体解决办法请见： 在 iOS 9 中运行企业版应用

在 iOS 9 中点击“安装”按钮后，没有弹出“是否安装”的提示？

这个问题是因为 iOS 9 的一个 Bug 导致的。出现这个问题的前提，一般是由于用户已经从苹果官方 App Store 上安装了相同的应用。解决办法是：先在设备中删除之前已经安装的应用，然后再重新安装即可。
为什么在 iOS 9 中，点击“安装”按钮后，没有任何反应，桌面也没有出现应用图标，但是状态栏上的网络图标在转？
这是由于 iOS 9 中的一个 Bug 造成的。虽然看上去没有反应，其实应用已经在后台开始下载并安装了，状态栏上的网络图标在转就是一个证明。这个时候，只要多等待一会儿就好了，应用安装完成之后会在桌面上显示出来的。


应用安装过程中提示"无法下载应用程序"
 
原因一：在导出 iOS App 的安装包文件（.ipa文件）时，选择了 Ad-hoc 方式，但是没有添加设备 UDID。

在导出 iOS 的安装包文件时，如果选择了 Ad-hoc 方式（一般用于苹果个人开发者账户），那么，如果要某台设备可以安装，则必须要将这台设备的 UDID 添加到导出安装包时所用的证书文件中（. mobileprovision文件），才可以在这台设备上安装。
原因三：在导出 iOS App 的安装包文件（.ipa文件）时，选择了 In-house 方式，但是证书已过期。

在导出 iOS 的安装包文件时，如果选择了 In-house 方式（一般用于苹果企业开发者账户），此时，如果出现无法安装的情况，开发者可以检查一下自己的企业开发者证书是否已过期。因为苹果对于企业开发者证书管理较为严格，所以开发者如果使用不当，可能会导致企业证书被封，被封后的企业证书导出的安装包，也是无法正确安装的。

原因四：开发者在生成App安装包时，没有在 Xcode 中设置正确的 Architecture。


iOS 应用的 Architecture（架构），决定了这款 iOS 应用可以在哪些设备机型上安装。例如，如果某个应用在 Xcode 中只添加了arm64 这一种 Architecture，那么最终打包后的安装包文件对于 iPad mini、iPhone5 等以下设备，都是无法安装的（因为这些设备都不是 arm64 架构）。换句话说，如果需要在某个设备上可以安装，App 就必须支持那个设备的 Architecture。
所以，正确的解决方法是，在生成 App 安装包时，尽可能让 App 支持更多的 Architecture。

具体操作方法是：在 Xcode - Build Settings - Architecture 中，增加 armv7、armv7s、arm64，以便所有设备都可以安装。然后，将 "Build active architecture only" 设置为 NO。对于各个 iOS 设备支持的 Architecture 类型。请点击这里查看。

原因五：App 支持的 iOS 系统版本，和当前设备系统版本不符。

App 支持的 iOS 系统版本过低或者过高，都可能导致 App 无法安装成功。例如，如果某个 App 设置了只支持 iOS 7.0 以上的系统时，那么，如果在 iOS 6.1 系统上安装时，肯定是无法安装成功。

因此，解决的方法也很简单，我们应该尽量让 App 尽可能支持更宽泛的系统版本。

具体操作方式是：在 Xcode - General - Deployment Info - Deployment Target 中，给 App 设置一个尽量低的版本，例如 iOS 5.0。

原因六：开发者上传的是一个破解的 ipa 安装包，或者是一个使用破解 Xcode 方式打包生成的 ipa 安装包，或者是通过 iTunes 生成的 ipa 安装包。


通过任何非 Xcode（或 Xcode 的命令行工具）生成的安装包，都是没有办法正确在设备上安装的（越狱设备除外）。常见的不正确的打包 ipa 的方式有：通过 iTunes 导出安装包文件、通过 iTools 导出安装包文件等等。
正确的方法是，使用一个正常的苹果开发者证书，通过未破解的 Xcode 打包生成 ipa 安装包。

原因七：设备上已经安装了这个App，且已经安装的 App 和要安装的 App 是用不同证书打包的。

这种情况下，也会造成 App 安装失败。解决的方式很简单，开发者只需将设备上原来已经安装的 App 删除，再重新安装新的 App 即可。

原因八：Info.plist 文件中的LSRequiresIPhoneOS 没有设置，或者设置了 NO。

对于 iOS 的 App 来说，如果Info.plist 文件中的LSRequiresIPhoneOS 没有设置，或者设置了 NO，那么由 Xcode 导出的安装包（.ipa 包），就不会包含 Payload 文件夹，而是被一个叫做 Applications 的文件夹代替。这样的安装包在安装时，会被 iOS 判定为无效的安装包，所以无法被正确安装。

解决方式也很简单，只需要将Info.plist 文件中的LSRequiresIPhoneOS 设置为 YES，然后重新打包即可。具体操作为：在 Xcode 中打开 Info.plist 文件，然后检查 LSRequiresIPhoneOS 是否已设置，如果没有设置，就添加一个，然后将LSRequiresIPhoneOS 的类型设置为 Boolean，值设置为 YES。

设置好以后，可以看到 Info.plist 文件中显示 Application requires iPhone environment 的值为 YES。

 
原因九：网络出现中断或异常。
遇到这种情况，用户可检查自己手机的所连接的网络是否稳定、速度是否正常等。可以尝试一下其他网站，或者更换一个 Wi-Fi，或者由 Wi-Fi 换成 3G/4G 等，然后重新安装。

在 iOS 8 上安装时，没有任何反应

这个是由于 iOS 8 的一个 bug 造成的，开发者可以尝试在plist中为bundle id加个后缀，详情可见 Stackoverflow 上的讨论。
安装 iOS 应用时，出现提示“无法连接到 ssl.pgyer.com”
这个问题一般是由于用户的网络，或者手机缓存错误造成的，可以尝试如下两个方法来解决：

重启手机，然后尝试重新安装。
换一个网络环境，例如换一个 Wi-Fi 热点，或由 Wi-Fi 换成 3G/4G 等，然后重新安装。

用这样的方式尝试后，一般都可以解决问题。 