---
title: XX视频图片上传小记
date: 2020-09-21 00:36:22
tags:
---

> 健忘越来越严重，记一下，以防失忆

###一. 业务操作流程

1.  约定图片尺寸

   * 大图：最长边不超过800px
   * 缩略图：最长边不超过400px

2. 操作流程
	1. 从相册选择图片
	
	2. 转存图片到本地（不要依赖相册选择器返回的路径，不准！）
	
	3. 调用SDK上传图片到腾讯云
	
	4. 拿到URL结果，替换域名
			把”cos.ap-guangzhou.myqcloud.com“ 替换成 ”image.myqcloud.com“，这样才有cdn加速效果
	
	5. 拼接下载规则
			连接后拼上 ”?imageView2/2/w/800/h/800“ ， 规定最长边不超过800px
		更多规则详见<https://cloud.tencent.com/document/product/460/6929>
	
	6. 拿到处理完的URL
			 http://rijvideo-resource-1251316161.image.myqcloud.com/047E82FC-1E01-4FD1-9CFA-21D2745F454C.GIF?imageView2/2/w/800/h/800

###二. 背景知识

1.  腾讯云SDK
    * SDK上传后返回的URL
        http://rijvideo-resource-1251316161.cos.ap-guangzhou.myqcloud.com/047E82FC-1E01-4FD1-9CFA-21D2745F454C.GIF

    * URL域名
        cos.ap-guangzhou.myqcloud.com
        
    * 服务器特点
        COS服务器，无加速，不可用数据万象图片处理功能
		
2. 数据万象
    * URL格式
        http://rijvideo-resource-1251316161.picgz.myqcloud.com/047E82FC-1E01-4FD1-9CFA-21D2745F454C.GIF

    * 域名
        picgz.myqcloud.com
        
    * 服务器特点
        数据万象服务器，无加速，可以使用数据万象图片处理功能
        
	* 腾讯云返回的URL，如何使用数据万象服务
		将SDK返回的URL替换域名即可，将URL中的"cos.ap-guangzhou.myqcloud.com"替换为"picgz.myqcloud.com"
			eg：
			替换域名前：http://rijvideo-resource-1251316161.cos.ap-guangzhou.myqcloud.com/047E82FC-1E01-4FD1-9CFA-21D2745F454C.GIF
			替换域名后：http://rijvideo-resource-1251316161.picgz.myqcloud.com/047E82FC-1E01-4FD1-9CFA-21D2745F454C.GIF
			
    * 数据万象图片处理文档
        https://cloud.tencent.com/document/product/460/6929
        
    * 使用举例
        图片缩放，最长边不超过100，应选择规则2，在原URL后拼接"?imageView2/2/w/100/h/100"
			原链接：http://rijvideo-resource-1251316161.picgz.myqcloud.com/047E82FC-1E01-4FD1-9CFA-21D2745F454C.GIF
			拼接结果：http://rijvideo-resource-1251316161.picgz.myqcloud.com/047E82FC-1E01-4FD1-9CFA-21D2745F454C.GIF?imageView2/2/w/100/h/100
		![原图](http://rijvideo-resource-1251316161.picgz.myqcloud.com/047E82FC-1E01-4FD1-9CFA-21D2745F454C.GIF)
		![缩放后](http://rijvideo-resource-1251316161.picgz.myqcloud.com/047E82FC-1E01-4FD1-9CFA-21D2745F454C.GIF?imageView2/2/w/100/h/100)
		
3. Cdn加速
    
    * URL格式
        http://rijvideo-resource-1251316161.image.myqcloud.com/047E82FC-1E01-4FD1-9CFA-21D2745F454C.GIF?imageView2/1/w/100/h/100

    * 域名
        image.myqcloud.com
    
    * 服务器特点
        数据万象服务器，有加速，可以使用数据万象图片处理功能

	* 如何使用
		将SDK返回的URL替换域名即可，将URL中的"cos.ap-guangzhou.myqcloud.com"替换为"image.myqcloud.com"
			eg：
			替换域名前：http://rijvideo-resource-1251316161.cos.ap-guangzhou.myqcloud.com/047E82FC-1E01-4FD1-9CFA-21D2745F454C.GIF
			替换域名后：http://rijvideo-resource-1251316161.image.myqcloud.com/047E82FC-1E01-4FD1-9CFA-21D2745F454C.GIF

###三. 关于上传的一些策略

1. 超时时间
    * wifi：30s
    * xg：60s
    
2. 分片策略
    * 1M一片
    * 最多并行4片

3. 断点重传
    * 基于分片，以分片为原子元素，上传失败整片开始重传


###三. 文档链接

1. 腾讯云SDK文档
https://cloud.tencent.com/document/product/436/11280

2. 数据万象接口文档
https://cloud.tencent.com/document/product/460/10511

（完）