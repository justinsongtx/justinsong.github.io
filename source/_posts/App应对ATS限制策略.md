---
title: App应对ATS限制策略
date: 2020-09-21 00:40:02
tags:
---




`结果先行：本次应对ATS，兴趣部落App选择接入TBS库，由TBS全局拦截HTTP请求，改为套接字的方式收发请求后返回给上层。`

## 问题背景


  ATS(App Transport Security),是苹果在ＷＷＤＣ 15提出的，Apple 在推进网络通讯安全的一个重要方式，按照苹果的要求非https的网络访问是被禁止的；当然现阶段我们可以通过在info.plist里面中添加 NSAppTransportSecurity 字典并且将 NSAllowsArbitraryLoads 设置为YES 来禁用 ATS。

  不过在WWDC 16中，Apple明确表示将收紧http的访问，从2017年1月1日起，所有提交的app默认不允许使用NSAllowsArbitraryLoads 来绕过ATS的限制，这样对于浏览器、手Q、空间等需要访问大量第三方http站点的应用来说，提前解决ATS的问题就迫在眉睫。

这一大推总结起来就是：app内不能再使用HTTP请求，否则有被下架的风险
## 有哪些解决办法？
* `替换app内所有http链接`，包涵所有app使用的链接和协议中下发的链接，并且要推动存储后台支持https协议－－－－－工作量较大，对存储后台有强依赖。

* `实现注册NSURLProtocol子类`，用代码实现拦截http请求，代码中自行请求数据，用套接字代替http请求－－－－－工作量巨大，安全性能无法预测。

* `使用TBS全局拦截http请求`，由TBS使用套接字请求数据后返回，上层完全无感知，手q等多款app已经接入，稳定至今－－－－－工作量小，无外部依赖，安装包增量23k左右，性能安全均优。

答案很明显：使用方案3，引入TBS

##对TBS的了解与考量

* `能否解决HTTP协议的限制`

      iOS– TBS通过接管webview的网络层请求，将请求通过SPDY协议发送到浏览器的后台代理服务器；后台代理服务器再通过骨干网络去对方服务器抓取数据；

    TBS的架构图如下：
![](http://km.oa.com/files/photos/pictures/201704/1492570441_6_w1560_h1066.png)


   浏览器使用的SPDY协议是基于底层socket自己实现的网络通信组件，没有使用任何Apple提供的网络组件，并且也不是http协议，所以能够完美解决ATS防止http的问题，并且SPDY协议本身是二进制协议，安全性强于HTTP；

* `安全性`
    * 解决运营商http网页劫持；TBS和浏览器后台代理之间是SPDY协议通道，运营商劫持代价很高，所以不会劫持；后台的代理服务器和网站的server之间是通过骨干网络访问也避免了运营商劫持；
    * 解决httpDNS的劫持；TBS和浏览器后台代理通过IP连接，HTTP请求到了后台的代理服务器才进行DNS解析，不给运营商DNS劫持机会；
    * spdy协议本身是支持加密的；相对http来说，安全性也是可以得到保障的；

* `速度`
    * 通过TBS和浏览器后台的长链接减少延时；
    * HTTP Header也压缩减少数据传输；
    * 并发网络请求提高效率；
    * 指定请求优先级加快网页展示；
    * 后台还做了就快接入、图片压缩、资源缓存、广告过滤；
    * spdy代理加速能力，在3G网络下网页访问速度有效提升 20% 到 30%；

* `安装包增量`
      安装包增量23k

* `性能`
      未发现性能问题
    
* `有哪些已经接入`
      现在已有手Ｑ、京东、微云、空间、now直播等APP接入了TBS

小结：经各方面考虑，目前接入TBS在解决ATS问题上是最优选择。

## 接入

* SDK加入工程（SDK地址<http://git.code.oa.com/QQBrowser_iOS/iOS_TBS_SDK>）
* 添加所需要的依赖
    * libz.tbd
    * libstdc++.tbd
    * WebKit.framework
* 实现QBProtocolWorker子类，实现`qbProxyProtocolCanInitWithRequest:isQBWebViewRequest:`函数，返回`YES`代表需要用SPDY协议代替当前的请求，反之返回`NO`。

    ```objectivec
    @implementation TBHttpProtocolWorker
    
    + (BOOL) qbProxyProtocolCanInitWithRequest:(NSURLRequest *)request isQBWebViewRequest:(BOOL)isQBWebViewRequest
    {
        if (isQBWebViewRequest) {
            return YES;
        }
        
        NSString* strUrl = [request.URL absoluteString];
        if ([[TBATSUtil sharedInstance] isATSActive] && [strUrl hasPrefix:@"http://"]) {
            return YES;
        }
        
        return NO;
    }
    
    @end
    ```
* 在app启动后注册我们的子类，开始监听http请求

    ```objectivec
    - (void)startTBSService
    {
        if (_isInit == NO && [self isATSActive])
        {
            [QBWebViewHelper startProxyWithWorker:[TBHttpProtocolWorker class]];
            _isInit = YES;
        }
    }
    ```
接入到此完成

##使用抓包工具验证

* 接入前

   ![](http://km.oa.com/files/photos/pictures/201704/1492570498_68_w682_h1248.png)

* 接入后
   ![](http://km.oa.com/files/photos/pictures/201704/1492570514_71_w680_h1236.png)

小结：绕过http请求，效果明显

(完)
