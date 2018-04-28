---
layout:     post
title:      "基于导向滤波实现边缘保留功能"
date:       2018-05-01
author:     "Hime"
header-img: "img/article-bg.JPG"
catalog: true
tags:
    - 算法
---

有这么一篇文章：
[Guided Image Filtering, by Kaiming He, Jian Sun, and Xiaoou Tang, in TPAMI 2013.](http://kaiminghe.com/publications/pami12guidedfilter.pdf)

根据其中的Guided Filter算法，将导向图设置为与输入图相同的图像，就会变成一种边缘保留的滤波器。

原公式则变为

![equation](http://latex.codecogs.com/gif.latex?m_{ij}=\frac{1}{(2r+1)(2r+1)}\sum_{k=i-r}^{i+r}\sum_{l=j-r}^{j+r}x_{kl})

![equation](http://latex.codecogs.com/gif.latex?v_{ij}=\frac{1}{(2r+1)(2r+1)}\sum_{k=i-r}^{i+r}\sum_{l=j-r}^{j+r}(x_{kl}-m_{ij})^2)

![equation](http://latex.codecogs.com/gif.latex?x^\prime_{ij}=(1-k)m_{ij}+kx_{ij})

其中

![equation](http://latex.codecogs.com/gif.latex?k=\frac{v_{ij}}{v_{ij}+\sigma})

而方差公式可由此获得

![equation](http://latex.codecogs.com/gif.latex?Var(x)=\bar{x^2_i}-(\bar{x_i})^2)

![equation](http://latex.codecogs.com/gif.latex?n_{ij}=\frac{1}{(2r+1)(2r+1)}\sum_{k=i-r}^{i+r}\sum_{l=j-r}^{j+r}x_{kl}^2 )

![equation](http://latex.codecogs.com/gif.latex?v_{ij}=n_{ij}-m_{ij}^2)

这样就可以快速取得方差值

以下是Metal的Shader，其中![equation](http://latex.codecogs.com/gif.latex?\sigma=0.01)

```c++
kernel void GuidedFilter(texture2d<float, access::read> inTexture [[texture(0)]],
                         texture2d<float, access::write> outTexture [[texture(1)]],
                         device float *input [[buffer(0)]],
                         uint2 gid [[thread_position_in_grid]])
{
    const short radius = input[0];
    const short size = (2 * radius + 1) * (2 * radius + 1);
    
    float4 mean = float4(0);
    float4 meanSquare = float4(0);
    
    for (short x = -radius; x <= radius; x++) {
        for (short y = -radius; y <= radius; y++) {
            float4 colorAtPixel = inTexture.read(ushort2(gid.x + x, gid.y + y));
            mean += colorAtPixel;
            meanSquare += colorAtPixel * colorAtPixel;
        }
    }
    mean /= size;
    meanSquare /= size;
    
    float4 v = meanSquare - mean * mean;
    float4 k = v / (v + 0.01);
    float4 fin = k * inTexture.read(gid) + (1.0 - k) * mean;
    
    outTexture.write(fin, gid);
}
```

下图为处理结果
