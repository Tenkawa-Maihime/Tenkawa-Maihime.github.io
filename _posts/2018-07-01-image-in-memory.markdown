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

施工中

图片所占用的内存与文件的大小无关，由图片的分辨率所决定。  
一张590KB大小，分辨率为2048x1536的图片，所占用的内存为  
2048 x 1536 x 4byte = 12MB  
当我们加载一张图片的时候，将会对其进行解码，并在内存中开辟空间存放以供渲染。

苹果有四种图片格式  
- Alpha 8
- luminance and alpha 8
- SRGB
- wide format
分别占用1、2、4、8个Byte
  
如果你画一张黑色图片，使用老的API：UIGraphicsBeginImageContextWithOptions，会为每个黑色像素分配4个Byte，但实际上只需要1个Byte就足够了。  
而使用这个新的API：UIGraphicsImageRenderer，从iOS12开始会自动选择图片格式，即自动为每个黑色像素分配1个Byte。这样子就能够节省75%的内存空间。

在我们日常现实图片的时候，往往并不会将整个图片1比1的现显示出来，而是显示一个缩略图。但实际使用UIImage的时候，会将图片解压到内存当中,并对其裁剪，
这样一来也是相当浪费内存。  
这时候可以才用ImageIO这一框架先对其进行降采样，它所分配的内存与裁剪后图像分辨率有关。
