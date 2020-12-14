---
layout: post
title: Generalizable Reconstruction
tags: [Computer Vision, 3D Shape Reconstruction, 2.5D Sketch]
gh-repo: xiumingzhang/GenRe-ShapeHD
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

这篇论文是在MarrNet基础上的工作，研究内容是物体3D形状重建，重点放在了可扩展性上，即从已知物体学习，结合一些先验知识，重建出训练集没见过的物体的3D形状，创新性比较大。

* 论文名称：[Learning to Reconstruct Shapes from Unseen Classes](https://arxiv.org/abs/1812.11166)

* 论文作者：Xiuming Zhang, Zhoutong Zhang, Chengkai Zhang, Joshua B. Tenenbaum, William T. Freeman, Jiajun Wu（MIT CSAIL, Google Research）

* 收录情况：NeurIPS 2018

![](../img/post/genre_fig2.png)

### 主要方法
利用单张图片的重建算法，学习一个函数 $f_{2D \rightarrow 3D}$，把图片中的2D物体映射成3D形状。本文通过正则化 $f_{2D \rightarrow 3D}$ 解决扩展性的问题。使用的正则化方法是把 $f_{2D \rightarrow 3D}$ 分解成 $geometric~projections$ 和 $learnable~reconstruction$ 模块。

本文的方法——$GenRe$，由3个可学习的模块组成，通过 $geometric~projections$ 模块连接。
* 第一个模块是单视图的 depth estimator $f_{2D \rightarrow 2.5D}$。输入是RGB图，输出是depth map，depth map可以理解成物体的可视化表面（因为获取深度的时候，计算的是相机到物体表面的深度）。因此，给定部分估计的深度图，重建问题变成了预测物体完整的表面。

* 因为物体3D表面很难形式化地表示出来（为什么这么说，因为物体表面不尽相同，不可能用一个方程完全表示，只能是选取一个函数作近似），这里使用球形图作近似（球形图，比较规则图形，方便用方程描述）。
    - $geometric~projections$ $p_{2.5D \rightarrow S}$ 把深度图转换成球形图，具体说是 $partial~ spherical~ map$
    - 然后把 $partial~ spherical~ map$ 输入到 $spherical~ map$修复网络，预测修复好的球形图（$inpainted~spherical ~map$）
    - 然后把 $inpainted~spherical ~map$ 投影到 voxel space

* 因为球形图仅有球形的外表面信息，不能处理沿球体半径的自遮挡，本文使用了 voxel refinement module解决这个问题。
    - 输入包含2部分
        * 一个是来自 $inpainted~spherical ~map$ 的体素图
        * 另一个是来自 $depth~map$ 的体素图
    - 输出最终的3D shape
