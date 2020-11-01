---
title: '记一次发布失败 ITMS-90034: Missing or invalid signature'
date: 2020-09-21 00:35:04
tags:
---

## 起因
今天项目提审，上传了ipa，叮咚，收到一封邮件，有不好的预感，果然，苹果爸爸又zuo妖了，提审失败

**解决办法：最终重做证书之后解决**

![enter image description here](http://km.oa.com/files/photos/pictures/202006/1591364435_57_w1528_h602.png)

## 经过
提示签名错误，查！
怎么查？

### 一. 从结果入手，查ipa里的签名

1. 解压ipa
2. 进入Payload/xxx.app/目录
3. 输入命令 

    ```
    security cms -D -i embedded.mobileprovision
    ```
4. 分析签名文件
![enter image description here](http://km.oa.com/files/photos/pictures/202006/1591364449_93_w1586_h1144.png)
![enter image description here](http://km.oa.com/files/photos/pictures/202006/1591364455_44_w2710_h704.png)
和keystore里的签名完全对应，而且没过期，未见异常

### 二. 从苹果邮件入手，上网搜索错误号“ITMS-90034”

网上解决方式有3种：
* 重新制作证书
* 删除过期证书
* 更改证书默认信任


## 结果

与蓝盾同事交涉，我们的证书已经是默认信任的，那就剩下**更新证书**了，来吧！

* 更新证书
* 重新打包
* 上传ipa
* 审核通过

一气呵成，回家吃饭！