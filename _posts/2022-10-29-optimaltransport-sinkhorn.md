---
layout: post
title: 最优传输理论-Sinkhorn算法
author: Pinzhi Wang
tags:
- Optimal Transport
- Sinkhorn
math: true
date: 2022-10-29 10:30 +0800
---

## 1 Optimal Transport
&emsp;&emsp;以一道题为例来解释**离散的最优传输问题**。令$\vec{r} = \left \[ 3,3,3,4,2,2,2,1 \right ]^{T}$表示每个人所获得的小吃的数量，$\vec{c} = \left [ 4,2,6,4,4 \right ]^{T} $记录每种小吃有多少份，则有如下定义：
<center>
\begin{align}
U(\vec{r},\vec{c}) & = \left \{
  \mathbf{P}\in\mathbb{R}^{n*m} | \mathbf{P}\vec{1}_{m}=\vec{r} , \mathbf{P}^{T}\vec{1}_{n}=\vec{c}  \tag {1}
\right  \}
\\ dim(\vec{r}) &= n = 8
\\ dim(\vec{c}) &= m = 5
\end{align}
</center>
$U(\vec{r},\vec{c})$包含所有可能的小吃分配方案，$\mathbf{P}$为我们要求的分配矩阵,每个用户对各小吃的喜爱程度存储在矩阵$\mathbf{M}\in\mathbb{R}^{n*m}$中（矩阵$\mathbf{M}$也被成为代价矩阵，本例中将矩阵$\mathbf{M}$中的元素取负即可得到其对应的代价矩阵）。
<br>最终问题可以表示为：
<center>
\begin{align}
d_\mathbf{M}(\vec{r},\vec{c})&=\min_{\mathbf{P}\in U(\vec{r},\vec{c}))}\sum {P_{ij}M_{ij}} \tag {2}
\end{align}
</center>
其中$d_\mathbf{M}(\vec{r},\vec{c})$又被称为 Wasserstein 距离。这里对矩阵$\mathbf{M}$取负表示希望分配时尽量能满足每个人的喜好从程度，如果不取负值则是表示按照每个人最不喜欢的方式分配，注意公式(2)min的含义。

## 2 Entropy Regularization
&emsp;&emsp;通过增加熵正则化可以求上述问题的近似解，并可以用Sinkhorn算法快速求解，上述问题的熵正则化可以定义为：
<center>
$$
 {d^\lambda}_\mathbf{M}(\vec{r},\vec{c})=\min_{\mathbf{P}\in U(\vec{r},\vec{c}))}\sum {P_{ij}M_{ij}}+\frac{1}{\lambda}(-P_{ij}(lnP_{ij}-1)) \tag {3}
$$
</center>
通过添加熵正则化约束，我们可以用Sinkhorn算法求解(3)，其中${d^\lambda}_\mathbf{M}(\vec{r},\vec{c})$也被称为Sinkorn距离。

## 3 Sinkhorn Alogrithm*
&emsp;&emsp;利用拉格朗日数乘法可以推导该算法公式，其中约束条件为$$ \mathbf{P}\vec{1}_{m}=\vec{r}  和  \mathbf{P}^{T}\vec{1}_{n}=\vec{c}$$，则公式推导如下：
<center>
\begin{align}
L(P_{ij},\alpha,\beta) &= \sum ({P_{ij}M_{ij}}+\frac{1}{\lambda}(-P_{ij}(lnP_{ij}-1)) + \vec{\alpha}^{T}(\mathbf{P}\vec{1}_{m}-\vec{r})+ \vec{\beta}^{T}(\mathbf{P}^{T}\vec{1}_{n}-\vec{c}) \tag {4}\\
\frac{\partial L(P_{ij},\alpha,\beta)}{\partial P_{ij}} &= M_{ij} - \frac{1}{\lambda}lnP_{ij}+\alpha_{i}+\beta_{j} =0 \tag {5}\\
P_{ij}&=e^{\lambda M_{ij}+\lambda \alpha_{i}+\lambda \beta_{j}} \tag{6}
\end{align}
</center>
$令K_{ij}=e^{\lambda M_{ij}}$，则$有P_{ij}=u_{i}K_{ij}v_{j}$，矩阵形式为：
<center>
$$
\mathbf{P}=diag(\vec{u})\mathbf{K} diag (\vec{v}) \tag{7}
$$
</center>
则可以由约束条件$$ \mathbf{P}\vec{1}_{m}=\vec{r} ，\mathbf{P}^{T}\vec{1}_{n}=\vec{c}$$和公式(7)推导出，$\odot$表示哈达玛积（Hadamard product）：
<center>
\begin{align}
diag(\vec{u})\mathbf{K} diag (\vec{v})\vec{1}_{m}&=\vec{r} \Rightarrow  \vec{u} \odot (\mathbf{K}\vec{v}) = \vec{r} \tag{8} \\
diag(\vec{v})\mathbf{K}^{T} diag (\vec{u})\vec{1}_{n}&=\vec{c} \Rightarrow  \vec{v} \odot (\mathbf{K}^{T}\vec{u}) = \vec{c} \tag{9} \\
\end{align}
</center>
**Sinkhorn算法通过迭代求解$\vec{u}和\vec{v}$，从而求解$\mathbf{P}$**，公式如下，**其中的除法指向量元素对应相除**：
<center>
\begin{align}
\vec{u}^{k+1}&=\vec{r} / (\mathbf{K}\vec{v}^{k}) \tag{10} \\
\vec{v}^{k+1}&=\vec{c} / (\mathbf{K}^{T}\vec{u}^{k+1}) \tag{11} \\
\end{align}
</center>
一般情况下，令$\vec{v}^{0}=\vec{1}_{m}$。
