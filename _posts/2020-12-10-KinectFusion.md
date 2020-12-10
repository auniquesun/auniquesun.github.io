---
layout: post
title: KinectFusion
tags: [Computer Vision, Surface Reconstruction, Geometry-Aware Interactions, Depth Cameras, GPU]
# gh-repo: jiajunwu/marrnet
# gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

* 论文名称：[KinectFusion: Real-time 3D Reconstruction and Interaction Using a Moving Depth Camera](https://jiajunwu.com/papers/marrnet_nips.pdf)

* 论文作者：Shahram Izadi, David Kim, Otmar Hilliges, David Molyneaux, Richard Newcombe, Pushmeet Kohli, Steve Hodges1, Dustin Freeman（Microsoft Research Cambridge）

* 收录情况：ACM symposium on User interface software and technology 2011

这篇论文是微软2011年发表的，研究内容是通过RGBD相机，实时重建3D场景表面并能和用户交互，后来产生了很大的影响力。其中一个重要的原因是在GPU上实现了
完整的软件+硬件系统，非常具有说服力。