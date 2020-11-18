---
layout: post
title: Total3DUnderstanding
tags: [Computer Vision, 3D Scene Understanding & Object Reconstruction, Joint Learning]
gh-repo: yinyunie/Total3DUnderstanding
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

这篇论文研究的问题是从单张RGB图片，运用深度学习的方法，同时建模室内布局，物体姿态和Mesh，取得了较好的效果。这篇文章是对其他工作的follow，
之前NeurIPS\ECCV出现过研究同样的文章，有必要同时看一下，弄清楚一些来龙去脉。

* 论文名称：[Total3DUnderstanding: Joint Layout, Object Pose and Mesh Reconstruction for Indoor Scenes from a Single Image](https://arxiv.org/abs/2002.12212)

* 论文作者：Yinyu Nie, Xiaoguang Han, Shihui Guo, Yujian Zheng, Jian Chang, Jian Jun Zhang（伯恩茅斯大学、港中文、深圳大数据研究所、厦大）

* 收录情况：ECCV 2018

### 主要方法
* 3D Object and Layout Estimation
    * 让世界坐标系和相机坐标系一致（潜台词说相机是静止的），y-axis 垂直于地面，x-axis 指向相机
        - 因此，相机姿态可以用$\textbf{R}(\beta, \gamma)$表示，$\beta$表示pitch角，$\gamma$表示roll角
        - 世界坐标系中，物体的3D bbox的中心点$\textbf{C} \in \mathbb{R}^3$，尺寸$\textbf{s} \in \mathbb{R}^3$，朝向$\theta \in [-\pi, \pi]$
        - 对于室内的物体（？？？为什么还分室内室外），3D中心$\textbf{C}$ 表示成像平面2D投影 $\textbf{c} \in \mathbb{R}^2$，物体到相机平面的距离 $d \in \mathbb{R}$
    * 给定相机内参矩阵$\textbf \mathbb{R}^{3 \times 3}$（原文把内参矩阵搞成了一个向量，写的不对），$\textbf{C}$ 可以表示成（？？？没懂推导）
        - $$ \textbf{C} = \textbf{R}^{-1}(\beta, \gamma) \cdot d \cdot \frac{\textbf{K}^{-1}[\textbf{c}, 1]^{T}}{{\parallel \textbf{K}^{-1}[\textbf{c}, 1]^{T} \parallel_{2}}$$
    * 2D 投影中心 $\textbf{c}$ 由两部分组成
        - $\textbf{c}^{b}$，bbox 中心点
        - $\textbf{\delta} \in \mathbb{R}^2$，偏移量
    * 从2D detection $\textbf{I}$到 3D box corners（？？？这些描述不太准确吧），网络进行了$\textbf{F}$变换
        - $$\textbf{F}(\textbf{I} | \textbf{\delta}, d, \beta, \gamma, \textbf{s}, \theta) \in \mathbb{R} ^{3 \times 8}$$
        - 3D Object Detection Network 预测 $(\textbf{\delta}, d, \textbf{s}, \theta)$
        - Layout Estimation Network 预测 $\textbf{R}(\beta, \gamma)$ 和 $(\textbf{C}, \textbf{s}^{l}, \theta^{l})$