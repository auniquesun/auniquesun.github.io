---
layout: post
title: OpenCV Learning Schedule
tags: [Computer Vision, OpenCV]
gh-repo: opencv/opencv
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

### 简介
OpenCV 是在[BSD许可证](https://opensource.org/licenses/BSD-3-Clause)下开源的计算机视觉和机器学习软件，经过专业社区长期开发和维护，在Windows、macOS、Linux、iOS、Android各大平台都有对应的实现，源代码用C++编写，通过模板化接口与STL容器无缝衔接，并提供了Python、Java、MATLAB编程接口，具有生态完备、功能齐全的特点，在学术界、工业界、各个组织得到了广泛应用。OpenCV是计算机视觉理论学习和工程实践不可回避的工具。

1. 几组数字：
    * 2500+ 计算机视觉相关算法
    * 47,000+ 用户组成的社区
    * 1800+ 万下载量

2. OpenCV能干什么？一些直观印象：
    * 检测和识别人脸、识别对象、对视频中的人类行为进行分类
    * 跟踪摄像机运动、跟踪运动对象
    * 提取物体的三维模型、从立体摄像机生成三维点云
    * 将图像拼接在一起以生成整个场景的高分辨率图像
    * 从图像数据库中找到相似的图像
    * 从使用flash拍摄的图像中去除红眼，跟踪眼球运动
    * 识别场景并建立标记以覆盖增强现实等

### 学习计划
从最近的机器感知课程可以感觉到，很多视觉任务，用不着复杂的模型实现，OpenCV提供的实现效果就很好，而且计算经过许多优化，很多情况下是解决视觉任务首选。由于官方文档内容太多，一个人学起来太耗时，我们成立一个学习小组，每人负责学习几个模块，实践其中几个视觉算法，然后把自己学的内容讲给大家。下面的 `tutorial` 分为4部分，每部分由一个同学负责

#### 版本
OpenCV 有很多版本，我们统一使用 [3.4.10](https://docs.opencv.org/3.4.10/d9/df8/tutorial_root.html)，对应的文档和代码也是这个版本

#### Tutorial I
* [Introduction to OpenCV](https://docs.opencv.org/3.4.10/df/d65/tutorial_table_of_content_introduction.html)
* [The Core Functionality (core module)](https://docs.opencv.org/3.4.10/de/d7a/tutorial_table_of_content_core.html)
* [Image Processing (imgproc module)](https://docs.opencv.org/3.4.10/d7/da8/tutorial_table_of_content_imgproc.html)
* [High Level GUI and Media (highgui module)](https://docs.opencv.org/3.4.10/d0/de2/tutorial_table_of_content_highgui.html)

#### Tutorial II
* [Image Input and Output (imgcodecs module)](https://docs.opencv.org/3.4.10/da/d8f/tutorial_table_of_content_imgcodecs.html)
* [Video Input and Output (videoio module)](https://docs.opencv.org/3.4.10/df/d2c/tutorial_table_of_content_videoio.html)
* [Camera calibration and 3D reconstruction (calib3d module)](https://docs.opencv.org/3.4.10/d6/d55/tutorial_table_of_content_calib3d.html)
* [Video analysis (video module)](https://docs.opencv.org/3.4.10/da/dd0/tutorial_table_of_content_video.html)

#### Tutorial III
* [2D Features framework (feature2d module)](https://docs.opencv.org/3.4.10/d9/d97/tutorial_table_of_content_features2d.html)
* [Object Detection (objdetect module)](https://docs.opencv.org/3.4.10/d2/d64/tutorial_table_of_content_objdetect.html)
* [Deep Neural Networks (dnn module)](https://docs.opencv.org/3.4.10/d2/d58/tutorial_table_of_content_dnn.html)

#### Tutorial IV
* [Machine Learning (ml module)](https://docs.opencv.org/3.4.10/d1/d69/tutorial_table_of_content_ml.html)
* [Computational photography (photo module)](https://docs.opencv.org/3.4.10/da/de7/tutorial_table_of_content_photo.html)
* [Images stitching (stitching module)](https://docs.opencv.org/3.4.10/d0/d33/tutorial_table_of_content_stitching.html)

#### Optional
* [GPU-Accelerated Computer Vision (cuda module)](https://docs.opencv.org/3.4.10/da/d2c/tutorial_table_of_content_gpu.html)
* [OpenCV iOS](https://docs.opencv.org/3.4.10/d3/dc9/tutorial_table_of_content_ios.html)
* [OpenCV Viz](https://docs.opencv.org/3.4.10/d7/df9/tutorial_table_of_content_viz.html)