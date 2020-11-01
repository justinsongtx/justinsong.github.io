---
title: ios知识梳理
date: 2020-09-07 00:04:17
tags:
---

# ios知识梳理

## 函数调用发生了什么事？
![15988664259962](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/09/07/15988664259962.jpg)


1. object_sendmsg 发送消息
2. 对象判空，根据不同返回值返回0 or nil
3. 用SEL先后在category的methodList，Class的cache， 类对象的methodList中查找IMP，查到就调用
4. 上面找不到就去superClass里找，实例方法在类对象里向上找，类方法在元类对象里向上找
5. 找到头还没找到就出发消息转发，有三次处理机会，动态方法解析，快速转发，完整转发，三个环节一个比一个消耗大。
     * 动态消息转发，给机会用runtime加个同名方法，返回yes就会调用新增的方法，返回no就crash了
     * 快速转发，转发给除自己以外的其他对象
     * 完整转发，这个比较麻烦，要先用SEL给方法做个签名，在做转发，优点是可以转发给多个对象，转发的方法也可以自定义，可以用于crash拦截，API版本兼容，实现oc伪多继承
6. 这都走完还没找到方法就抛出异常

待补充：
* 参数压栈出栈，存在哪个寄存器
* 类方法的快速转发和完整转发
* 多继承如何实现
* 数据的存储区


知识链接：
https://juejin.im/post/6844903600968171533
https://www.jianshu.com/p/f900de4a1495

## KVO底层原理

runtime增加一个子类，动态修改isa指针，将对象指向子类，在子类中使用CoreFoundation中的预埋方法重写setXXX方法，在setXXX前后调用BeforeValueChange和AfterValueChange,记录值的变化，然后通知监听者。

## 内存分配

![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/09/06/15994047841974.jpg)

程序启动时操作系统会给程序分配一段内存，用于存放程序运行中所需的数据，按存储用途，地址由低到高分为：**代码区**、**常量区**、**全局静态区**、**堆**、**栈**，其中全局静态区分为BSS区和数据区，常量区和全局静态区在有些文章里合称全局数据区。

* 代码区，存放可执行程序的二进制代码

* 常量区，存放静态变量，static，const，字面量
    ```
    void main(){
        char array[] = "aaa"; // "aaa"存放在栈区的array[]内存上
        char *point = "aaa"; // "aaa"存放在常量区
    }
    ```

* 全局静态区
    * BSS区，未初始化的全局变量
    * 数据区，已初始化的全局变量

* 堆
    由程序员控制，存放程序运行中，动态分配的内存块，大小不固定，扩展时由地地址向高地址扩展。
    
* 栈
     存放函数的参数，临时变量。

* 地址
    调试时，指针的值，0x10xxx的就是全局数据区，0x60xxx的就是堆区，0x7fxxxxxx的就是栈区
    

## 内存管理

* 参考
    * https://www.jianshu.com/p/48665652e4e4
    * https://www.jianshu.com/p/7bd2f85f03dc  (牛逼)
    
* MRC
    手动引用计数，谁申请谁释放，但有的时候不知道什么时候释放，例如下面例子，函数不能release返回的对象，也不知道何时该release
    
```
    - （NSString *）newName {
        NSString *name = [NSString alloc] init];
        name = @"123";
        return name;
    }
```

* 自动释放池
    针对上面的场景，ios引入了一个新的角色——自动释放池，不知道什么时候该释放的时候可以通过调用autorelease，将对象放入自动释放池，当自动式方池销毁的时候，会向池子中所有的对象发送一条release消息，这时如果对象引用计数变为0，则系统会将对象释放，否则就内存泄露了。
    
    调用autorelease把对象放入自动释放池不会增加引用计数。  
```
Person *p = [Person new];
p = [p autorelease];
NSLog(@"count = %lu", [p retainCount]); // 计数还为1
```
    * autorelease的创建方法
        * 使用NSAutoreleasePool来创建
        
        ```
        NSAutoreleasePool *autoreleasePool = [[NSAutoreleasePool alloc] init];
        Person *p = [[[Person alloc] init] autorelease];
        [autoreleasePool drain];
        ```
       
        * 使用@autoreleasepool创建    
        
        ```
        @autoreleasepool
{ // 创建一个自动释放池
        Person *p = [[Person new] autorelease];
        // 将代码写到这里就放入了自动释放池
} // 销毁自动释放池(会给池子中所有对象发送一条release消息)
        
        ```

* RunLoop和AutoreleasePool的关系

    主线程的NSRunLoop在监测到事件响应开启每一次event loop之前，会自动创建一个autorelease pool，并且会在event loop结束的时候执行drain操作，释放其中的对象。

* Thread和AutoreleasePool的关系

    包括主线程在内的所有线程都维护有它自己的自动释放池的堆栈结构。新的自动释放池被创建的时候，它们会被添加到栈的顶部，而当池子销毁的时候，会从栈移除。对于当前线程来说，Autoreleased对象会被放到栈顶的自动释放池中。当一个线程线程停止，它会自动释放掉与其关联的所有自动释放池。

* 主线程自动释放池
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/09/07/15994159260554.jpg)

    1. 程序启动到加载完成后，主线程对应的RunLoop会停下来等待用户交互
    2. 用户的每一次交互都会启动一次运行循环，来处理用户所有的点击事件、触摸事件。
    3. RunLoop检测到事件后，就会创建自动释放池;
    4. 所有的延迟释放对象都会被添加到这个池子中;
    5. 在一次完整的运行循环结束之前，会向池中所有对象发送release消息，然后自动释放池被销毁;

* ARC
    自动引用计数，在新申请的对象前面默认加入_autorelease
    
* 解决循环引用
    * 主动剪短引用循环，使用弱引用
    * 使用Xcode静态分析
    * 使用Insurment中的leak，查看引用循环图
    * 使用MLLeakFinder
    
    
## RunLoop
* 资料（牛逼）
    https://blog.ibireme.com/2015/05/18/runloop/
* RunLoop包含的5种Mode（模式）
    * 初始化模式
    * 默认模式
    * UI模式
    * 占位模式
    * 内核事件模式
只有默认模式和UI模式能用到，其他都是屁，看看就行

* 每个Mode中有三种对象
    * Observer
    * Source
    * Timer

* 执行流程
    ![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2020/09/07/15994818051599.jpg)
    
* 底层实现
    除了装逼，没其他用，不计了
    
## 事件响应
苹果注册了一个 Source1 (基于 mach port 的) 用来接收系统事件，其回调函数为 __IOHIDEventSystemClientQueueCallback()。

当一个硬件事件(触摸/锁屏/摇晃等)发生后，首先由 IOKit.framework 生成一个 IOHIDEvent 事件并由 SpringBoard 接收。这个过程的详细情况可以参考这里。SpringBoard 只接收按键(锁屏/静音等)，触摸，加速，接近传感器等几种 Event，随后用 mach port 转发给需要的App进程。随后苹果注册的那个 Source1 就会触发回调，并调用 _UIApplicationHandleEventQueue() 进行应用内部的分发。

_UIApplicationHandleEventQueue() 会把 IOHIDEvent 处理并包装成 UIEvent 进行处理或分发，其中包括识别 UIGesture/处理屏幕旋转/发送给 UIWindow 等。通常事件比如 UIButton 点击、touchesBegin/Move/End/Cancel 事件都是在这个回调中完成的。


## 手势识别
当上面的 _UIApplicationHandleEventQueue() 识别了一个手势时，其首先会调用 Cancel 将当前的 touchesBegin/Move/End 系列回调打断。随后系统将对应的 UIGestureRecognizer 标记为待处理。

苹果注册了一个 Observer 监测 BeforeWaiting (Loop即将进入休眠) 事件，这个Observer的回调函数是 _UIGestureRecognizerUpdateObserver()，其内部会获取所有刚被标记为待处理的 GestureRecognizer，并执行GestureRecognizer的回调。

当有 UIGestureRecognizer 的变化(创建/销毁/状态改变)时，这个回调都会进行相应处理。


## 界面更新
当在操作 UI 时，比如改变了 Frame、更新了 UIView/CALayer 的层次时，或者手动调用了 UIView/CALayer 的 setNeedsLayout/setNeedsDisplay方法后，这个 UIView/CALayer 就被标记为待处理，并被提交到一个全局的容器去。

苹果注册了一个 Observer 监听 BeforeWaiting(即将进入休眠) 和 Exit (即将退出Loop) 事件，回调去执行一个很长的函数：
_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv()。这个函数里会遍历所有待处理的 UIView/CAlayer 以执行实际的绘制和调整，并更新 UI 界面。

## iOS 为什么必须在主线程中操作UI


因为UIKit不是线程安全的。试想下面这几种情况：

1. 两个线程同时设置同一个背景图片，那么很有可能因为当前图片被释放了两次而导致应用崩溃。

2. 两个线程同时设置同一个UIView的背景颜色，那么很有可能渲染显示的是颜色A，而此时在UIView逻辑树上的背景颜色属性为B。

3. 两个线程同时操作view的树形结构：在线程A中for循环遍历并操作当前View的所有subView，然后此时线程B中将某个subView直接删除，这就导致了错乱还可能导致应用崩溃。
iOS4之后苹果将大部分绘图的方法和诸如 UIColor 和 UIFont 这样的类改写为了线程安全可用，但是仍然强烈建议讲UI操作保证在主线程中执行。

## CALayer

在iOS当中，所有的视图都从一个叫做UIVIew的基类派生而来，UIView可以处理触摸事件，可以支持基于Core Graphics绘图，可以做仿射变换（例如旋转或者缩放），或者简单的类似于滑动或者渐变的动画。

CALayer类在概念上和UIView类似，同样也是一些被层级关系树管理的矩形块，同样也可以包含一些内容（像图片，文本或者背景色），管理子图层的位置。它们有一些方法和属性用来做动画和变换。和UIView最大的不同是CALayer不处理用户的交互。CALayer并不清楚具体的响应链。

UIView和CALayer是一个平行的层级关系，每一个UIView都有一个CALayer实例的图层属性，也就是所谓的backing layer，视图的职责就是创建并管理这个图层，以确保当子视图在层级关系中添加或者被移除的时候，他们关联的图层也同样对应在层级关系树当中有相同的操作。实际上这些背后关联的Layer图层才是真正用来在屏幕上显示和做动画，UIView仅仅是对它的一个封装，提供了一些iOS类似于处理触摸的具体功能，以及Core Animation底层方法的高级接口。

UIView 的 Layer 在系统内部，被维护着三份同样的树形数据结构，分别是：

* 图层树（这里是代码可以操纵的，设置属性的最终值会立刻在这里更新）；

* 呈现树（是一个中间层，系统就在这一层上更改属性，进行各种渲染操作。比如一个动画是更改alpha值从0到1，那么在逻辑树上此属性会被立刻更新为最终属性1，而在动画树上会根据设置的动画时间从0逐步变化到1）；

* 渲染树（其属性值就是当前正被显示在屏幕上的属性值）；


## iOS事件处理与图像渲染（牛逼）
https://www.cnblogs.com/yulang314/p/5091894.html

## weak实现

https://www.jianshu.com/p/3c5e335341e0


## 默认属性
* 对应基本数据类型，默认关键字为
atomic, assign, readwrite

* 对应对象类型，默认关键字为
atomic, strong, readwrite

*  atomic 和 nonatomic区别
https://www.jianshu.com/p/7288eacbb1a2


## Block原理
https://www.jianshu.com/p/221d0778dcaa

## YUV
https://glumes.com/post/ffmpeg/understand-yuv-format/

## I帧B帧P帧
https://www.cnblogs.com/yongdaimi/p/10676309.html

## H264
https://www.cnblogs.com/Lxk0825/p/9925041.html

## 直播开发大全
https://www.jianshu.com/p/bd42bacbe4cc

## 20分钟OpenGL
https://zhuanlan.zhihu.com/p/56693625

## 自研直播协议（牛逼 详细）
https://yq.aliyun.com/articles/668499

## WebRTC协议
https://blog.csdn.net/moyebaobei1/article/details/86703258

## HLS协议和RTMP协议
https://cloud.tencent.com/developer/article/1509053

## H264(比较全)
https://www.jianshu.com/p/0c296b05ef2a

## App启动耗时优化
* 启动时间 = pre-main耗时+main耗时
* pre-main阶段优化：
    * 删除无用代码
    * 抽象重复代码
    * +load方法做的事情延迟到initialize中，或者+load的事情不宜花费太多时间
    * 减少不必要的framework，或者优化已有framework
* Main阶段优化
    * didFinishLauchingwithOptions里代码延后执行
    * 首次启动渲染的页面优化

## crash防护
* unrecognized selector crash
    https://www.jianshu.com/p/6a12e9d92366
* KVO crash
    https://www.jianshu.com/p/e3713d309283
* NSNotification crash
    https://www.cnblogs.com/Xylophone/p/6394056.html
* NSTimer crash
    https://www.cnblogs.com/Xylophone/p/6394076.html
* Container crash（数组越界，插nil等）
* NSString crash （字符串操作的crash）
* Bad Access crash （野指针）
* UI not on Main Thread Crash (非主线程刷UI (机制待改善))

## 内存泄露问题
主要集中在循环引用问题中，如block、NSTime、perform selector引用计数问题。
* 使用Xcode静态分析
* 使用Insurment中的leak，查看引用循环图
* 使用MLLeakFinder

## UI卡顿优化
https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/

* CPU优化
    * 对象懒加载
    * Time Profile找出大量耗时的操作
    * 布局计算，cell(缓存高度，后台线程提前计算布局)，文本布局计算（后台提前计算）
    * 视图创建消耗（cell复用）
    * 减少自动布局的使用
    * 图片解码（后台线程提前解码成Bitmap，不要用JPEG，用PNG）
* GPU优化
    * 图片的渲染，tableview里面大量图片（可以将多张图片合成一张，或使用低清晰度的缩略图）
    * 减少视图混合（减少UIview层级）
    * 减少离屏渲染，（CALayer 的 border、圆角、阴影、mask都会触发离屏渲染，可以将需要显示的效果在后台绘制成一张图片）
    
## GPUImage源码


## Opengl ES

https://juejin.im/post/6844903843180838920