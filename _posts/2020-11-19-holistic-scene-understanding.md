---
layout: post
title: Holistic Scene Understanding
tags: [Computer Vision, 3D Scene Understanding & Object Reconstruction, Joint Learning]
gh-repo: thusiyuan/cooperative_scene_parsing
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

* 论文名称：[Cooperative Holistic Scene Understanding: Unifying 3D Object, Layout, and Camera Pose Estimation](https://arxiv.org/abs/1810.13049)

* 论文作者：Siyuan Huang, Siyuan Qi, Yinxue Xiao, Yixin Zhu, Ying Nian Wu, Song-Chun Zhu（来自UCLA）

* 收录情况：NeurIPS 2018

### 简介
虽然人类能比较容易的感知一个3D场景（**200ms内完成**；Potter, 1975, 1976, Schyns and Oliva, 1994, Thorpe et al., 1996），但是3D场景的整体理解在计算机视觉研究中是个基础又有挑战性的问题，主要的困难是：单张RGB图片包含大量带有歧义的3D信息。3D场景整体理解包含几项任务：

* 相机3D姿态估计。RGB图片来自相机，搞清楚相机在哪个位置，从哪个角度拍摄了照片，有助于提升2D图片和3D场景的一致性 $consistency$。
* 场景3D布局估计。通常指室内场景，相机3D姿态与场景3D布局组合起来，能反映场景的全局几何性质 $global ~geometry$。
* 场景中物体3D检测。反映的是局部信息 $local~ details$

但是一些已知的方法或者不够高效，或者解决了部分问题
* Traditional Methods
    * Abhinav Gupta, Martial Hebert, Takeo Kanade, and David M Blei. Estimating spatial layout of rooms using volumetric reasoning about objects and surfaces. In Conference on Neural Information Processing Systems (NIPS), 2010
    * Yibiao Zhao and Song-Chun Zhu. Image parsing with stochastic scene grammar. In Conference on Neural Information Processing Systems (NIPS), 2011
    * Wongun Choi, Yu-Wei Chao, Caroline Pantofaru, and Silvio Savarese. Understanding indoor scenes using 3d geometric phrases. In IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2013
    * Yinda Zhang, Shuran Song, Ping Tan, and Jianxiong Xiao. Panocontext: A whole-room 3d context model for panoramic scene understanding. In European Conference on Computer Vision (ECCV), 2014
    * Hamid Izadinia, Qi Shan, and Steven M Seitz. Im2cad. In IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2017
    * Siyuan Huang, Siyuan Qi, Yixin Zhu, Yinxue Xiao, Yuanlu Xu, and Song-Chun Zhu. Holistic 3d scene parsing and reconstruction from a single rgb image. In European Conference on Computer Vision (ECCV), 2018
    * Alexander G Schwing, Sanja Fidler, Marc Pollefeys, and Raquel Urtasun. Box in the box: Joint 3d layout and object reasoning from single images. In IEEE International Conference on Computer Vision (ICCV), 2013
* Deep Learning
    * 

### 主要方法
