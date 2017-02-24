---
layout: post
title: 3D动画
tags: 笔记
---

CALayer层的3D动画简单学习

##几种3D动画的简介

```Objective-C
UIImageView *img = [[UIImageView alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
img.layer.transform = CATransform3DMakeRotation(M_PI, 1, 0, 0);//3D动画，围绕x轴旋转
img.layer.transform = CATransform3DMakeTranslation(100, 50, 0);//移动到x为100，y为50的地方
img.layer.transform = CATransform3DMakeScale(0.5, 0.5, 0);//放大缩小
```

>在使用的时候可以给个1秒的动画效果

```Objective-C
    //可以给个动画的效果
    [UIView animateWithDuration:1
                     animations:^{
                         img.layer.transform = CATransform3DMakeRotation(M_PI, 1, 0, 0);//3D动画，围绕x轴旋转
                         img.layer.transform = CATransform3DMakeTranslation(100, 50, 0);//移动到x为100，y为50的地方
                         img.layer.transform = CATransform3DMakeScale(0.5, 0.5, 0);//放大缩小
                     }];//组合变化
```
