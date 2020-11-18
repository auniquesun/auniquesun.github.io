---
layout: post
title: Survey - 3D Scenes Modeling
tags: [Computer Vision, CNN, 3D Scenes Modeling]
# gh-repo: cvlab-dresden/DSAC
# gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

3D场景建模有着广阔的研究和应用前景，这里的场景既包括室内又包括室外。最近几年大量基于深度学习的方法涌现出来，取得了一些惊艳的效果，促进了该领域的发展。本文关注点是3D场景建模领域的一些重要文献，并把它们整理出来，简要介绍每篇论文的研究背景、研究方法、主要贡献，让读者对3D场景建模的问题、方法有大致了解。在后几期的博客中，会逐一挑选这方面有代表性的论文，详细讲解给大家。

### Structured3D
* 论文名称：Structured3D: A Large Photo-realistic Dataset for Structured 3D Modeling

* 论文作者：Jia Zheng, Junfei Zhang, Jing Li, Rui Tang, Shenghua Gao, and Zihan Zhou（酷家乐公司、上海技术大学）

* 收录情况：ECCV 2020

* 简介
    1. 近年来，研究人员对于开发基于学习的方法进行3D场景建模和理解兴趣越来越大。这些方法通常会检测和使用显著性半全局、全局结构，常用的结构有：线、面、立方体、平滑表面、各种类型的对称形状。但是3D场景建模的真实标签需要人工标注，这通常是不够高效的，人工标注大量数据也是很有挑战性的。

    2. 本文制作了一个**合成数据集**（synthetic dataset）—— Structured3D，它提供了大规模带有3D结构标注信息的图片，能用于很多结构化3D建模任务。数据集的制作用到了公司内部专业的室内设计工具，抽取场景的3D结构。

    3. Structured3D 高质量图片的生成有工业级渲染引擎生成。这些生成的图片和真实的图片组合起来，训练深度神经网络来估计房间布局，实验表明效果得到提升。

### PlaneRecover
* 论文名称：Recovering 3D Planes from a Single Image via Convolutional Neural Networks

* 论文作者：Fengting Yang and Zihan Zhou（宾夕法尼亚州立大学）

* 收录情况：ECCV 2018

* 简介
    1. 本文研究的问题是：从单张人造环境图片，恢复3D表面

    2. 提出了 plane structure-induced loss 来训练深度神经网络，预测平面分割图（plane segmentation map）和3D平面的参数，证明训练深度神经网络恢复3D平面是可行的

    3. 为了避免打量人工标注过程，训练网络时没有显式利用3D平面标注信息，而是利用大规模RGB-D数据集，并且用数据集中的语义标签对平面/非平面做了准确分类。

    4. 实验证明本文的3D表面重建方法超越了已知方法，为基于视觉的导航和人机交互提供了新思路。

### Total3DUnderstanding
* 论文名称：Total3DUnderstanding: Joint Layout, Object Pose and Mesh Reconstruction for Indoor Scenes from a Single Image

* 论文作者：Yinyu Nie, Xiaoguang Han, Shihui Guo, Yujian Zheng, Jian Chang, Jian Jun Zhang（伯恩茅斯大学、港中文、深圳大数据研究所、厦大）

* 收录情况：ECCV 2018

* 简介
    1. 室内场景语义重建涉及到：（1）场景理解（2）物体重建

    2. 本文认为**已知的方法**或者解决了其中部分问题，或者把重点放在独立的物体

    3. 本文提出一个端到端的神经网络联合重建室内布局、物体边界框、网格（？？？mesh）。整个方法建立在一个整体的scene context，提出一种又粗到细的由三个组件构成的等级结构：
        1. 带有相机姿态的房间布局
        2. 3D object bounding boxes
        3. object meshes

    4. 本文认为理解每个组件有助于其他组件的理解，并为场景理解和重建赋能。在SUN RGB-D、Pix3D数据集的实验表明本文的方法在以下方面优于其他
        * indoor layout estimation
        * 3D object detection
        * mesh reconstruction

* **我的一些疑问与思考**    
    * 这篇论文是做3D场景理解和物体重建的，那么3D场景理解具体指什么？是把场景中的每类物体识别出来，并分割吗？物体重建要如何重建？我知道必须的步骤是识别出物体，重建时候尽可能和原物体相同
    * Mesh Reconstruction 是指什么？Voxel Representation 是什么？
    * 理解了这些基本概念，我有什么思路做这个事情？
    * 设计怎样的实验？
    * 用什么数据集？如何评价实验效果？
    * 做视觉研究展示效果，画怎样的图，用什么画图？

### KinectFusion
* 论文名称：KinectFusion: Real-time 3D Reconstruction and Interaction Using a Moving Depth Camera

* 论文作者：Shahram Izadi, David Kim, Otmar Hilliges, David Molyneaux, Richard Newcombe, Pushmeet Kohli, Jamie Shotton, Steve Hodges, Dustin Freeman, Andrew Davison, Andrew Fitzgibbon（微软剑桥研究院、帝国理工大学、纽卡斯特大学、兰卡斯特大学、多伦多大学）

* 收录情况：ACM symposium on User interface software and technology. 2011

* 简介
    1. KinectFusion使得用户手持一个 Kinect 相机，来回移动，就能快速建立细节清楚的室内3D场景图。

    2. 仅来自Kinect的深度数据，被用来实时地跟踪传感器的3D姿态，和重构几何图形。

    3. 本文展示了地城北的手持扫描、几何感知的增强现实和基于物理原理的交互系统。传感器使用GPU进行物体分割、用户交互没有降低相机跟踪和重建的性能。

    4. 一些扩展功能被用于实时 multi-touch 交互，可触摸的平面/非平面重建