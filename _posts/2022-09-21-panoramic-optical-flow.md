---
layout: post
title: 全景光流定义
author: Pinzhi Wang
tags:
- OpticalFlow
math: true
date: 2022-09-21 17:00 +0800
---

&emsp;&emsp;本篇博客主要介绍全景光流的定义，这里的定义是本人自己结合学长的ECCV文章中的全景深度定义，自己总结出的定义方式。

## 1 Cubemap
&emsp;&emsp;Cubemap（立方体贴图），在Unity中的定义为一个由六个独立的正方形纹理组成的集合，它将多个纹理组合起来映射到一个单一纹理。而在本文，其六个面并不是纹理而是六个照相机（FOV均为90）的六个画面，其中相机所在位置相同，相机的角度不同，分别拍摄立方体六个面不同画面（front right back left up down）。
<div  align="center">
  <img src="{{site.url}}/assets/cubemap.png">
</div>

## 2 从Cubemap光流到全景光流
&emsp;&emsp;因为一般cubemap六个面的光流是在**各个相机的图像坐标系**的图像坐标系下的uv方向上的像素变换值，本文将将不同图像坐标系下的像素变化转换到**以六个相机为中心的球体坐标系**下角度的变化，球的半径R为cubemap图像宽度/长度的一半，换言之，**该球体是cubemap形成正方体的内切球**。<br>
&emsp;&emsp;设从Cubemap形成的正方体$C$上的一像素点$P$与正方体的内切球$S$的中心$O$形成的向量$\vec{OP}$，与平面$YOZ$的夹角为$\varphi$；设点$P$在平面$YOZ$上的投影为点$P'$，$\vec{OP'}$与$\vec{OZ}$的夹角设为$\theta$。全景光流的变化即为$\theta和\varphi$的角度变化，我们可以通过如下方式进行求解。<br>

### 2.1 $\vec{OY}$方向的转换
<div  align="center">
  <img src="{{site.url}}/assets/panoramic_optical_flow/trans_OY.jpg">
</div>

### 2.2 $\vec{OY}$反方向$\vec{OY'}$的转换
<div  align="center">
  <img src="{{site.url}}/assets/panoramic_optical_flow/trans_OY'.jpg">
</div>
