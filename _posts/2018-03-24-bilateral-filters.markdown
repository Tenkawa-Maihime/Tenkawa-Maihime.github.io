---
layout:     post
title:      "双边滤波"
date:       2018-03-24
author:     "Hime"
header-img: "img/article-bg-2015.jpg"
catalog: true
tags:
    - 算法
---

双边滤波其实和均值滤波，高斯滤波没有多大的区别。  
它只是在这些滤波器的基础上对边缘区域进行加权处理，从而减少模糊对边缘的影响。
对于非边缘区域，我们给滤波器一个较大的权重，以提高其模糊效果，在边缘区域则给一个较小的权重来保留细节。

双边滤波的具体应用？当然是美颜啦~

下面是Metal的Shader，为了方便计算，采用了均值模糊

```c++
kernel void BilateralFilter(texture2d<float, access::read> inTexture [[texture(0)]],
                            texture2d<float, access::write> outTexture [[texture(1)]],
                            device float *input [[buffer(0)]],
                            uint2 gid [[thread_position_in_grid]])
{
    const short radius = input[0];
    const float spacial = 1.0 / (2 * radius + 1) / (2 * radius + 1);
    const float coe = -0.5 / pow(0.3, 2);
    
    float weightSum = 0;
    float3 colorSum = float3(0, 0, 0);
    const float4 colorAtCenter = inTexture.read(gid);
    
    for (short x = -radius; x <= radius; x++) {
        for (short y = -radius; y <= radius; y++) {
            
            ushort2 pixel = ushort2(gid.x + x, gid.y + y);
            float4 colorAtPixel = inTexture.read(pixel);
            
            float closeness = exp(pow(distance(colorAtPixel.xyz, colorAtCenter.xyz), 2) * coe);
            float weight = closeness * spacial;
            
            colorSum += colorAtPixel.rgb * weight;
            weightSum += weight;
        }
    }
    outTexture.write(float4((colorSum / weightSum), 1), gid);
}
```

另一种加权实现，速度更快，但效果不佳

```c++
float closeness = 1 - distance(colorAtPixel.xyz, colorAtCenter.xyz) / length(float3(1,1,1));
```

下图为处理结果  
第一行为均值模糊  
第二行为双边模糊  
从左至右分别为原图，以及卷积核大小为5x5、9x9、13x13、17x17。
可以在新标签页中打开图片查看原图

![](/img/bilateral/Beauty.jpg)




