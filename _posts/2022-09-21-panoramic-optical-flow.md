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
&emsp;&emsp;Cubemap（立方体贴图），在Unity中的定义为一个由六个独立的正方形纹理组成的集合，它将多个纹理组合起来映射到一个单一纹理。而在本文，其六个面并不是纹理而是六个照相机（FOV均为90度）的六个画面，其中相机所在位置相同，相机的角度不同，分别拍摄立方体六个面不同画面（front right back left up down）。
<div  align="center">
  <img src="{{site.url}}/assets/cubemap.png">
</div>

## 2 从Cubemap光流到全景光流
<div  align="center">
  <img src="{{site.url}}/assets/panoramic_optical_flow/overview.jpg">
</div>
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

### 2.3 $\vec{OZ}$方向的转换
<div  align="center">
  <img src="{{site.url}}/assets/panoramic_optical_flow/trans_OZ.jpg">
</div>

### 2.4 $\vec{OZ}$反方向$\vec{OZ'}$的转换
<div  align="center">
  <img src="{{site.url}}/assets/panoramic_optical_flow/trans_OZ'.jpg">
</div>

### 2.5 $\vec{OX}$方向的转换
<div  align="center">
  <img src="{{site.url}}/assets/panoramic_optical_flow/trans_OX.jpg">
</div>

### 2.6 $\vec{OX}$反方向$\vec{OX'}$的转换
<div  align="center">
  <img src="{{site.url}}/assets/panoramic_optical_flow/trans_OX'.jpg">
</div>

## 3 关键代码实现思想
&emsp;&emsp;代码采用Python语言实现，关键函数为calculate_angle_difference，该函数计算每个像素对应的$\bigtriangleup\theta$和$\bigtriangleup\varphi$。
{% highlight python %}
def calculate_angle_difference(cubemap_list, cubemap_h=1024):

    # init calculate parameters
    base_u = np.arange(cubemap_h*cubemap_h,dtype=np.float32)
    base_u = np.mod(base_u,cubemap_h)
    base_u.resize((cubemap_h,cubemap_h))
    base_v = base_u.T
    center_u = center_v = (cubemap_h-1)/2
    diff_u_center_u = base_u - center_u
    diff_v_center_v = base_v - center_v

    phi_list = []
    theta_list = []

    for i in range(6):
        diff_u_center_u_offset = diff_u_center_u+cubemap_list[i][...,0]
        diff_u_center_u_offset[diff_u_center_u_offset==0.0]= 1e-5 # 防止除0
        diff_v_center_v_offset = diff_v_center_v+cubemap_list[i][...,1] # 防止除0
        diff_v_center_v_offset[diff_v_center_v_offset==0.0]= 1e-5
        phi = None
        theta =None

        if i == 0:# 处理cubemap 1 OZ正方向数据
            phi  = - np.arccos((-diff_u_center_u)/(np.sqrt(center_v*center_v+diff_u_center_u*diff_u_center_u+diff_v_center_v*diff_v_center_v)))\
            + np.arccos((-diff_u_center_u_offset)/(np.sqrt(center_v*center_v+diff_u_center_u_offset*diff_u_center_u_offset+diff_v_center_v_offset*diff_v_center_v_offset)))
            theta = - (diff_v_center_v/np.abs(diff_v_center_v))*np.arccos((center_v)/(np.sqrt(center_v*center_v+diff_v_center_v*diff_v_center_v)))\
            + (diff_v_center_v_offset/np.abs(diff_v_center_v_offset))*np.arccos((center_v)/(np.sqrt(center_v*center_v+diff_v_center_v_offset*diff_v_center_v_offset)))
        elif i==1: # 处理cubemap 2 OX反方向数据
            phi  = - np.arccos((-center_v)/(np.sqrt(center_v*center_v+diff_u_center_u*diff_u_center_u+diff_v_center_v*diff_v_center_v)))\
            + np.arccos((-center_v)/(np.sqrt(center_v*center_v+diff_u_center_u_offset*diff_u_center_u_offset+diff_v_center_v_offset*diff_v_center_v_offset)))
            theta = - (diff_v_center_v/np.abs(diff_v_center_v))*np.arccos((-diff_u_center_u)/(np.sqrt(diff_u_center_u*diff_u_center_u+diff_v_center_v*diff_v_center_v)))\
            + (diff_v_center_v_offset/np.abs(diff_v_center_v_offset))*np.arccos((-diff_u_center_u_offset)/(np.sqrt(diff_u_center_u_offset*diff_u_center_u_offset+diff_v_center_v_offset*diff_v_center_v_offset)))
        elif i==2: # 处理cubemap 3 OZ 反方向数据
            phi  = - np.arccos((diff_u_center_u)/(np.sqrt(center_v*center_v+diff_u_center_u*diff_u_center_u+diff_v_center_v*diff_v_center_v)))\
            + np.arccos((diff_u_center_u_offset)/(np.sqrt(center_v*center_v+diff_u_center_u_offset*diff_u_center_u_offset+diff_v_center_v_offset*diff_v_center_v_offset)))
            theta = - (diff_v_center_v/np.abs(diff_v_center_v))*np.arccos((-center_v)/(np.sqrt(center_v*center_v+diff_v_center_v*diff_v_center_v)))\
            + (diff_v_center_v_offset/np.abs(diff_v_center_v_offset))*np.arccos((-center_v)/(np.sqrt(center_v*center_v+diff_v_center_v_offset*diff_v_center_v_offset)))
        elif i==3: # 处理cubemap 4 OX 正方向数据
            phi  = - np.arccos((center_v)/(np.sqrt(center_v*center_v+diff_u_center_u*diff_u_center_u+diff_v_center_v*diff_v_center_v)))\
            + np.arccos((center_v)/(np.sqrt(center_v*center_v+diff_u_center_u_offset*diff_u_center_u_offset+diff_v_center_v_offset*diff_v_center_v_offset)))
            theta = - (diff_v_center_v/np.abs(diff_v_center_v))*np.arccos((diff_u_center_u)/(np.sqrt(diff_u_center_u*diff_u_center_u+diff_v_center_v*diff_v_center_v)))\
            + (diff_v_center_v_offset/np.abs(diff_v_center_v_offset))*np.arccos((diff_u_center_u_offset)/(np.sqrt(diff_u_center_u_offset*diff_u_center_u_offset+diff_v_center_v_offset*diff_v_center_v_offset)))
        elif i==4: # OY正方向数据
            phi  = - np.arccos((-diff_u_center_u)/(np.sqrt(center_v*center_v+diff_u_center_u*diff_u_center_u+diff_v_center_v*diff_v_center_v)))\
            + np.arccos((-diff_u_center_u_offset)/(np.sqrt(center_v*center_v+diff_u_center_u_offset*diff_u_center_u_offset+diff_v_center_v_offset*diff_v_center_v_offset)))
            theta = np.arccos((diff_v_center_v)/(np.sqrt(center_v*center_v+diff_v_center_v*diff_v_center_v))) \
            - np.arccos((diff_v_center_v_offset)/(np.sqrt(center_v*center_v+diff_v_center_v_offset*diff_v_center_v_offset)))
        elif i==5: # OY反方向数据
            phi  = - np.arccos((-diff_u_center_u)/(np.sqrt(center_v*center_v+diff_u_center_u*diff_u_center_u+diff_v_center_v*diff_v_center_v)))\
            + np.arccos((-diff_u_center_u_offset)/(np.sqrt(center_v*center_v+diff_u_center_u_offset*diff_u_center_u_offset+diff_v_center_v_offset*diff_v_center_v_offset)))
            theta = - np.arccos((-diff_v_center_v)/(np.sqrt(center_v*center_v+diff_v_center_v*diff_v_center_v))) \
            + np.arccos((-diff_v_center_v_offset)/(np.sqrt(center_v*center_v+diff_v_center_v_offset*diff_v_center_v_offset)))

        theta[theta>PI] -= 2*PI #theta角的限制
        theta[theta<-PI]+= 2*PI

        phi_list.append(phi)
        theta_list.append(theta)

    return phi_list, theta_list
{% endhighlight %}
&emsp;&emsp;这样我们可以得到$H\times   W\times 2$的图像。最后可以采用Cubemap转erp投影的方式，该方法是从cubemap中采样RGB值到投影图上，
本文的全景光流转换则是采样$\bigtriangleup\theta$和$\bigtriangleup\varphi$。
{% highlight python %}
        # cube map data [back down front left right up]
        input_cube = np.zeros((6,2,cubemap_h,cubemap_h),dtype=np.float32)
        input_cube[0,0,...] = phi_list[2]
        input_cube[0,1,...] = theta_list[2]
        input_cube[1,0,...] = phi_list[5]
        input_cube[1,1,...] = theta_list[5]
        input_cube[2,0,...] = phi_list[0]
        input_cube[2,1,...] = theta_list[0]
        input_cube[3,0,...] = phi_list[3]
        input_cube[3,1,...] = theta_list[3]
        input_cube[4,0,...] = phi_list[1]
        input_cube[4,1,...] = theta_list[1]
        input_cube[5,0,...] = phi_list[4]
        input_cube[5,1,...] = theta_list[4]

        batch = torch.tensor(input_cube,dtype=torch.float32)
        out = cube2erp.ToEquirecTensor(batch, 'nearest')
        out = out.numpy()
        #print(out)
        out = out * cubemap_h /PI
{% endhighlight %}

## 4 关于$\bigtriangleup\theta$和$\bigtriangleup\varphi$的限制
&emsp;&emsp;设平面$YOZ$的单位法向量为$\vec{n}=[1,0,0]^{T}$，也就是说它的方向与$\vec{OX}$相同，从而我们可得$\varphi \in [-\frac{\pi }{2},\frac{\pi }{2} ]$；同样地，我们可知$\theta \in [-\pi,\pi]$，定义以$\vec{OX}$为旋转轴，在左手坐标系下（大拇指方向与$\vec{OX}$相同），$\vec{OZ}$顺时针旋转为正（向上旋转），逆时针旋转（向下旋转）为负，也就是说，$\theta$的符号与$\vec{OP}$的$Y$分量的符号相同。素在不同cubemap上的光流即可被转换到角度$\varphi$和$\theta$的变换$\bigtriangleup\varphi$和$\bigtriangleup\theta$，值得注意的是$\bigtriangleup\varphi$的大小不会超过$−\pi$和$\pi$，而$\bigtriangleup\theta$的大小需要限制到$ [-\pi,\pi]$之内，因为我们假定两帧之间的物体位移很小，i.e.当$\bigtriangleup\theta$大于$\pi$时， $\bigtriangleup\theta -= 2\pi$；当$\bigtriangleup\theta$小于$-\pi$时，$\bigtriangleup\theta += 2\pi$，下面是对该限制的具体举例。<br>
<div  align="center">
  <img src="{{site.url}}/assets/panoramic_optical_flow/theta_restrict.jpg">
</div>

## 5 Carla虚拟数据集中全景光流可视化和相邻帧之间的关键点映射检验
### 5.1 Carla光流数据处理
&emsp;&emsp;经过本人实验验证和对网上资源的总结，carla采集到的光流需要经过前处理，从而转变为计算机视觉中标准的光流定义。设从carla中得到的光流为$Flow_{origin}:(H,W,2)$，cubemap的高度/宽度(cubemap是正方形)为$H$，转换后的光流为$Flow_{out}$，则有：<br>
<div  align="center">
$$
\begin{eqnarray}
F_{out}[:,:,0] & = & F_{origin}[:,:,0]*(-H/2) \\
F_{out}[:,:,1] & = & F_{origin}[:,:,1]*(H/2)
\end{eqnarray}
$$
</div>
下面是以front面为例的代码:
{% highlight python %}
front  = np.load(os.path.join(r'./sensor_raw_data','l_camof1',file))
front[:,:,0] *= -512.0 # u方向 width方向 width=height=1024
front[:,:,1] *= 512.0 # v方向 height方向
{% endhighlight %}
&emsp;&emsp;最终得到的光流是后向光流（BackwardFlow），设当前帧图像像素点$P_{cur}$，前一帧图像像素点$P_{last}$，当前帧相对于上一帧的光流变化$(\bigtriangleup u,\bigtriangleup v)$，则有：<br>
<div  align="center">
$$
\begin{eqnarray}
P_{last}-P_{cur}=(\bigtriangleup u,\bigtriangleup v)
\end{eqnarray}
$$
</div>

### 5.2 全景光流可视化图片展示
<div  align="center">
  <img src="{{site.url}}/assets/panoramic_optical_flow/opticalflow_vis.jpg">
</div>
### 5.3 相邻帧之间的关键点映射检验
&emsp;&emsp;本小节展示通过第2367帧的采集光流，将第2367帧中的部分点与第2365帧中的点进行匹配，以此大致检验光流的正确性。
<div  align="center">
  <img src="{{site.url}}/assets/panoramic_optical_flow/images_match_eval.jpg">
</div>
