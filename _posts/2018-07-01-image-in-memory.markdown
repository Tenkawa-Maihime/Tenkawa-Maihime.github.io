---
layout:     post
title:      "图片内存优化"
date:       2018-07-01
author:     "Hime"
header-img: "img/article-bg.JPG"
catalog: true
tags:
    - 优化
---

图片所占用的内存与文件的大小无关，由图片的分辨率所决定。  
一张590KB大小，分辨率为2048x1536的图片，所占用的内存为  
2048 x 1536 x 4byte = 12MB  
当我们加载一张图片的时候，会对其进行解码，并在内存中开辟空间存放以供读取渲染。

苹果有四种图片渲染格式  
- Alpha 8
- luminance and alpha 8
- SRGB
- wide format
分别占用1、2、4、8个Byte
  
如果你画一张黑色图片，使用老的API：UIGraphicsBeginImageContextWithOptions，会为每个黑色像素分配4个Byte，但实际上只需要1个Byte就足够了。  
而使用这个新的API：UIGraphicsImageRenderer，从iOS12开始会自动选择图片格式，即自动为每个黑色像素分配1个Byte。这样子就能够节省75%的内存空间。

在我们日常现实图片的时候，往往并不会将整个图片1比1的展示出来，而是显示一张缩略图。但实际使用UIImage的时候，会将图片完整解压到内存当中，再对其裁剪，这样一来也是相当浪费内存。  
这时候可以采用ImageIO这一底层框架先对其进行降采样，它所分配的内存只与裁剪后图像分辨率大小有关。

在FastImageCache这一框架中，它将解压后的图片存入闪存中，以减少内存的使用。

另外，在APP退到后台的时候，图片所占用的内存依然存在，苹果官方建议将你所看不见的图片给释放掉，需要时再重新创建。


