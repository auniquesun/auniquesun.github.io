---
layout: post
title: Airport Birds Dataset
tags: [Dataset, Object Detection, Computer Vision]
gh-repo: tzutalin/labelImg
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

### 简介
1. 从机场部署的摄像头获取视频数据，视频包含飞鸟活动的场景，从中得到飞鸟图片。已有视频大都是冬天录制的，相对春夏秋季，冬天飞鸟数量偏少。
    - 2020.11
    - 2020.12
    - 2021.01

2. 经过算法过滤，初步得到 18,000+ 包含飞鸟的图片，这些图片已经有了飞鸟标注，目前的工作是人工检查这些图片
    1. 修正不准确的标注
    2. 补全未标注的飞鸟
    3. 删除标错的标注

### 流程
1. 共 18,000+ 图片，每人分配大约3000张，会把数据文件 `data_xxx.zip` 拷给大家

3. 解压数据后，进入`data`目录，结构如下所示
    ```
    data/
         images/
         labels/
         classes.txt
    ```

4. 把`labels`目录下的所有文件、`classes.txt`文件拷入`images`目录

1. 人工检查标注信息时，使用 [labelImg](https://github.com/tzutalin/labelImg) 软件，打开链接根据指南安装（推荐 miniconda3 方式安装）
    * step 1：打开软件图形界面时，进入`labelImg`目录，运行如下命令
        ```
        python labelImg.py
        ```
    * step 2：人工检查，从`labelImg`图形界面选择`images`目录打开
        1. 修正已有矩形框
        2. 新建矩形框
        3. 删除矩形框
        
        - **保存格式选择 YOLO**
        - `labelImg`一些常用快捷键
            * d $\rightarrow$ next
            * a $\rightarrow$ prev
            * w $\rightarrow$ create a rectangle box
            * del $\rightarrow$ delete the selected rectangle box