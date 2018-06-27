---
layout:     post
title:      "图片内存优化"
date:       2018-06-17
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

苹果的这个API：UIGraphicsBeginImageContextWithOptions，会为图像的每个像素分配4个Byte。  
而iOS10后的这个API：UIGraphicsImageRenderer，从iOS12开始会自动选择图片格式。  
如果你画一张黑色图片，老API会为每个黑色像素分配4个Byte，但实际上只需要1个Byte就足够了。
