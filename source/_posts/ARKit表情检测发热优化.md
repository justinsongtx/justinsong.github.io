---
title: ARKit表情检测发热优化
date: 2022-05-13 11:40:57
tags:
---
# ARKit 表情检测发热优化

# 背景

使用ARKit检测表情时时长发烫，找了一些优化办法，记在这里

# 优化1：降低帧率和分辨率

帖子：

[https://developer.apple.com/forums/thread/704304?answerId=710540022#710540022](https://developer.apple.com/forums/thread/704304?answerId=710540022#710540022)

ARKit会提供几种不同的videoFormat, 其中包含30帧 60帧，1080p和720p采集，选择30帧和720p对降低能耗有很大作用。

```swift
let configuration = ARFaceTrackingConfiguration()
for format in ARFaceTrackingConfiguration.supportedVideoFormats {

    // Use the framesPerSecond and imageResolution properties of the format to make your selection.
    if selectionCriteriaIsMet {
        configuration.videoFormat = format
        break
    }
}

session.run(configuration)
```

# 优化2：关闭不需要的开关

做表情检测时不需要world track，那就不要开worldTrackingEnabled；不需要光源数据就关掉lightEstimationEnabled；不需要音频数据就关掉providesAudioData；不需要额外的检测如人像分割、人体dect，就关掉frameSemantics。只做表情检测，不需要的其他开关一律关掉, 如下代码

```swift
// 初始化配置
ARFaceTrackingConfiguration *config = [[ARFaceTrackingConfiguration alloc] init];
// 选择30帧&最小分辨率
config.videoFormat = [self getMinVideoFormat];
// 限制检测人脸数
config.maximumNumberOfTrackedFaces = 1;
// 关掉光照估计，默认开
config.lightEstimationEnabled = NO;
// anchor位置从摄像头视角出发
config.worldAlignment = ARWorldAlignmentCamera;
// 关掉音频数据，默认关
config.providesAudioData = NO;
// 关掉世界跟踪，默认关
config.worldTrackingEnabled = NO;
// 关掉帧检测（人像分割，人体检测等等），默认关
config.frameSemantics = ARFrameSemanticNone;
```

# 优化3：干掉ARSCNView

感谢gchiste大神的帖子，才让我找到了干掉ARSCNView的办法，帖子：

[https://developer.apple.com/forums/thread/100403?answerId=354629022#354629022](https://developer.apple.com/forums/thread/100403?answerId=354629022#354629022)

[https://developer.apple.com/forums/thread/113797](https://developer.apple.com/forums/thread/113797)

```swift
import ARKit

class ViewController: UIViewController, ARSessionDelegate {
   
    let session = ARSession()
   
    override func viewDidLoad() {
        super.viewDidLoad()
       
        session.run(ARFaceTrackingConfiguration())
        session.delegate = self
       
    }
   
   
    func session(_ session: ARSession, didAdd anchors: [ARAnchor]) {
        print(anchors)
    }
   
    func session(_ session: ARSession, didUpdate anchors: [ARAnchor]) {
        for anchor in anchors {
            if let faceAnchor = anchor as? ARFaceAnchor {
                print(faceAnchor.blendShapes[ARFaceAnchor.BlendShapeLocation.mouthClose])
            }
        }
    }
}
```