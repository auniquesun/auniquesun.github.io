---
layout: post
title: Learning 3D Semantic Scene Graph
tags: [Computer Vision, 3D Semantic Scene Graph, Indoor Reconstruction]
# gh-repo: opencv/opencv
# gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

* 论文名称：[Learning 3D Semantic Scene Graphs from 3D Indoor Reconstructions](https://jiajunwu.com/papers/marrnet_nips.pdf)

* 论文作者：Johanna Wald, Helisa Dhamo, Nassir Navab, Federico Tombari（TUM、Google Docs）

* 收录情况：CVPR 2020

### 简介
3D场景理解就是感知和理解用3D数据表示的场景，不仅要求能识别和定位出3D空间的物体，而且能理解物体间的关系和所处的环境。这种彻底的3D场景理解对很多任务具有吸引力，比如自动导航，AR、VR。

当前的3D场景理解包含了实例分割、语义分割、3D物体检测等任务。这些工作把重点放在物体语义上，物体之间的关系和物体所处的环境主要用来提升物体分类的准确性。

最近，根据图像的场景理解开始探讨场景图的用途，比如用它来辅助理解物体之间的关系。在此之前，场景图已经被用在计算机图形学，排列图形场景的空间表示，其中节点表示物体实例，边表示连个节点的相对变换。这些概念被成功扩展到计算机视觉数据集，有 support structures/ semantic relationships and attributes / hierarchy mapping of scene entities。
**场景图在图片搜索、图像生成方面展现出用处**。

本文想要集中研究3D场景图的语义和潜在用处，目标是获得带标签的实例节点、带语义的关系边的场景图（如下所示），这种场景图具有以下功能：

* 对于场景和噪声的变化比较鲁棒  
* 填补文本、图像数据类型差异带来的鸿沟，在2D-3D场景检索、VQA方面很有应用前景
![](../img/post/3dssg_fig1.png)