---
title: CVPixelBufferRef 常用函数
date: 2021-03-16 15:07:49
tags:
---
# 常用函数

## 加锁解锁

```
CVPixelBufferLockBaseAddress(pixelBuffer, 0);
CVPixelBufferUnlockBaseAddress(pixelBuffer, 0);
```

## 引用释放

```
CFRetain(pixelBuffer);
CFRelease(pixelBuffer);
```

## 获取通颜色道个数

```
int planeCount = CVPixelBufferGetPlaneCount(pixelBuffer);
```

## 获取颜色空间种类

```
/**基本只有kCVPixelFormatType_32BGRA和kCVPixelFormatType_32ABGR*/
OSType type = CVPixelBufferGetPixelFormatType(pixelBuffer);
```

## 获取宽高

```
int width = (int)CVPixelBufferGetWidth(pixelBuffer);
int height = (int)CVPixelBufferGetHeight(pixelBuffer);
```

## 获取buffer数据地址

```
unsigned char* srcBuffer = (unsigned char*)(CVPixelBufferGetBaseAddress(pixelBuffer));
```

## 获取数据长度

```
/** 这个接口获取的是完整的pixelbuffer数据长度，其中包含对齐像素的长度，有可能会比width * height * channel的结果大，多出来的就是对齐像素所占的字节数*/
int srcLen = CVPixelBufferGetDataSize(pixelBuffer);
```

## 获取单行所占字节数

```
    CVPixelBufferLockBaseAddress(bgraPB, 0);

    int bufferWidth = (int)CVPixelBufferGetWidth( bgraPB );
    int bufferHeight = (int)CVPixelBufferGetHeight( bgraPB );
    
    /**单行所占字节数，其中包含对齐像素的大小*/
    size_t srcRowBytes = CVPixelBufferGetBytesPerRow( bgraPB );
    /**每个像素所占的字节数*/
    long srcPixelBytes = srcRowBytes / bufferWidth;
    /**整个buffer所占字节数，其中包含对齐像素的大小*/
    long srcLen = CVPixelBufferGetDataSize(bgraPB);
    
    CVPixelBufferUnlockBaseAddress( bgraPB, 0 );
```

## 创建CVPixelBufferRef

```
/**Core Video并不支持RGBA格式，常用的支持有BGRA和ARGB，其他支持的格式查阅：https://developer.apple.com/library/archive/qa/qa1501/_index.html*/
+ (CVPixelBufferRef)createPixelBuffer:(size_t)width height:(size_t)height formatType:(OSType)formatType
{
    CVPixelBufferRef pixel_buffer;
    CFDictionaryRef empty; // empty value for attr value.
    CFMutableDictionaryRef attrs;
    
    // our empty IOSurface properties dictionary
    empty = CFDictionaryCreate(kCFAllocatorDefault
                               , NULL
                               , NULL
                               , 0
                               , &kCFTypeDictionaryKeyCallBacks
                               , &kCFTypeDictionaryValueCallBacks);
    attrs = CFDictionaryCreateMutable(kCFAllocatorDefault
                                      , 1
                                      , &kCFTypeDictionaryKeyCallBacks
                                      , &kCFTypeDictionaryValueCallBacks);
    CFDictionarySetValue(attrs, kCVPixelBufferIOSurfacePropertiesKey, empty);

    CVReturn err = CVPixelBufferCreate(kCFAllocatorDefault, width, height, formatType, attrs, &pixel_buffer);
    if (err) {
        NSLog(@"failed");
    }
    CFRelease(attrs);
    CFRelease(empty);
    
    return pixel_buffer;
}
```

## CVPixelBufferRef转NSImage

```
/** pixelBuffer 转 NSImage*/
+ (NSImage *)imageFromPixelBuffer:(CVPixelBufferRef)pixelBuffer {
    CIImage *ciImage = [CIImage imageWithCVPixelBuffer:pixelBuffer];
    CIContext *temporaryContext = [CIContext contextWithOptions:nil];
    
    CGRect frame = CGRectMake(0, 0, CVPixelBufferGetWidth(pixelBuffer), CVPixelBufferGetHeight(pixelBuffer));
    CGImageRef videoImage = [temporaryContext createCGImage:ciImage fromRect:frame];

    NSImage *image = [[NSImage alloc] initWithCGImage:videoImage size:NSZeroSize];
    CGImageRelease(videoImage);

    return image;
}
```

# 坑点


* 背景

    MacOS下CVPixelBuffer直接转纹理转成的纹理target是GL_TEXTURE_RECTANGLE，需要再转换成通用的GL_TEXTURE_2D纹理。于是想到把buffer取出来自己上传。

* 现象

    CVPixelBufferRef取出buffer上传纹理之后，结果出现扭曲，逐行查找发现是`CVPixelBufferGetBytesPerRow`返回的数据多了4，这个API应该返回一行像素点所占用的字节数，比如一个4通道，宽高为240*450的图片，一行的字节数应该是4 * 240 = 960, 而实际情况是`CVPixelBufferGetBytesPerRow`返回了964，比宽度多了4个字节，也就是多了一个像素。

* 原因
    
    一番搜索后，发现`CVPixelBufferGetBytesPerRow`返回的是屏幕真实显示的像素个数，有时会有对齐像素，导致获取的字节数不准确，可以知道这个api 与 width * channel是不等效的，不要用这个api来组装vImage_Buffer，mac和ios设备都会有这种对其像素。

* 解决

    我的解决办法是用计算出对齐像素的个数，逐行memcpy排除掉这些像素，然后在转NSImage或上传纹理。 如果有更好的方法请留言~

一行代码胜千言，下面是我的CVPixelBuffer剔除对齐像素的代码：

```

+ (NSData *)rgbaBufferFromBGRAPixel:(CVPixelBufferRef)bgraPB {
    
    CVPixelBufferLockBaseAddress(bgraPB, 0);
    
    int bufferWidth = (int)CVPixelBufferGetWidth( bgraPB );
    int bufferHeight = (int)CVPixelBufferGetHeight( bgraPB );
    
    /**源*/
    size_t srcRowBytes = CVPixelBufferGetBytesPerRow( bgraPB );
    long srcPixelBytes = srcRowBytes / bufferWidth;
    long srcLen = CVPixelBufferGetDataSize(bgraPB);
    uint8_t *srcBuffer = (uint8_t *)CVPixelBufferGetBaseAddress( bgraPB );
    
    /**目标*/
    long destPixelBytes = 4;
    size_t destRowBytes = bufferWidth * destPixelBytes;
    long destLen = destRowBytes * bufferHeight;
    uint8_t *destBuffer = (uint8_t *)malloc(destLen);
    
    /**逐行拷贝, 跳过无用的对齐像素，否则图像会扭曲*/
    for ( int row = 0; row < bufferHeight; row++ )
    {
        uint8_t *srcAddress = srcBuffer + (row * srcRowBytes);
        uint8_t *destAddress = destBuffer + (row * destRowBytes);
        memcpy(destAddress, srcAddress, destRowBytes);
    }
    
    CVPixelBufferUnlockBaseAddress( bgraPB, 0 );
    
    NSData *rgbaData = [[NSData alloc] initWithBytes:destBuffer length:destLen];
    free(destBuffer);
    
    return rgbaData;
}
```
记一篇Stack Overflow：
https://stackoverflow.com/questions/44179891/avplayeritemvideooutput-copypixelbufferforitemtime-gives-incorrect-cvpixelbuffer/44211003#44211003