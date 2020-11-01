# 记一次bug，微信登录分享跳转多次

> 前几周遇到的问题，记了个标题，到今天才有时间填坑，来，整！

项目使用蓝盾平台打包，用通配符企业证书冲签名，微信SDK更新`1.8.6.2`之后，出现一个问题，微信登录和分享时，会在微信app和业务app之前，反复跳转两次，这个问题在本地调试是没有的。
<img src="https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/06/21/15927539305083.jpg" width="40%">


### 先看看微信SDK文档

![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/06/21/15927524709966.jpg)

让我们检查Universal Link，说明是UL没生效，用苹果的[验证官网](https://link.jianshu.com/?t=https://search.developer.apple.com/appsearch-validation-tool/)检查，UL是正常的，且本地调试没问题，排除代码问题。

### 再看看打包平台

看看蓝盾平台，和证书最相关的就是`iOS重签名插件`
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/06/21/15927530005449.jpg)

嗯？插件有个UL字段，看起来问题出在这，看看蓝盾爸爸的[文档](https://iwiki.oa.tencent.com/pages/viewpage.action?pageId=17506192)，看看这字段具体描述。

![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/06/21/15927531842392.jpg)

![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/06/21/15927531962045.jpg)

果然，蓝盾爸还要求在插件中重新设置一遍UL，不然项目中的UL字段会被通配符企业证书覆盖为空，导致UL功能失效，微信跳转的时候找不到我们注册的UL，所以有了上面的问题。

但是，插件只支持单个target的场景，多target场景不支持，能打包成功，但手机装不上，然而我们项目就是多target的，咦，这眼泪是怎么回事。

### 多target工程怎么办？

蓝盾爸爸的文档中提供了两种解决办法：

1. 别用UL了，放弃治疗
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/06/21/15927535655877.jpg)

2. 使用公共构建机
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/06/21/15927537681684.jpg)

接入公共构建机，打包出来ipa，验证微信分享，一切正常，解决！

（完）