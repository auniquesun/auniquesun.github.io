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
    * 晴天、阴天、上午、下午、雨夹雪

2. 经过算法过滤，初步得到 18,000+ 包含飞鸟的图片，这些图片已经有了飞鸟标注，目前的工作是人工检查这些图片
    1. 修正不准确的标注
    2. 补全未标注的飞鸟
    3. 删除标错的标注

### 流程
1. 共 18,000+ 图片，每人分配大约3000张，会把数据文件 `data_xxx.zip` 拷给大家

3. 解压数据后，进入`data_xxx`目录，结构如下所示
    ```
    data_xxx/
             sub_dir_1/
             sub_dir_2/
             ...
    ```
    * 每个`sub_dir`包含3类文件：png图片、txt标签、classes.txt。其中一张png图片对应一个txt标签文件，classes.txt包含物体类别，本数据集只有bird类
        - classes.txt格式
            * class_name

        - txt标签文件格式（**同YOLO**）：每行1个物体，用5元组表示
            ```
            class_id obj_x_center/img_width obj_y_center/img_height obj_width/img_width obj_height/img_height
            ```

1. 人工检查标注信息时，使用 [labelImg](https://github.com/tzutalin/labelImg) 软件，打开链接根据指南安装（推荐 miniconda3 方式安装）
    * step 1：预设物体类别，本数据集只有一类物体——bird，进入`labelImg`目录，修改`data/predefined_classes.txt`内容为：bird，同时在拷给大家的数据文件目录下，建立`classes.txt`文件，内容为：bird。
    * step 2：打开软件图形界面时，在`labelImg`目录，运行如下命令
        ```
        python labelImg.py
        ```
    * step 3：人工检查，从`labelImg`图形界面选择`images`目录打开
        1. 修正已有矩形框
        2. 新建矩形框
        3. 删除矩形框
        
        - **保存格式选择 YOLO**
        - `labelImg`一些常用快捷键
            * d $\rightarrow$ next
            * a $\rightarrow$ prev
            * w $\rightarrow$ create a rectangle box
            * del $\rightarrow$ delete the selected rectangle box

1. **设置检查点**：图片量比较大，不可能一次性标完，每标一批设置一个检查点，例如

    {: .center-block :}
    | 第几次 | 文件夹 | 结束图片 |
    | :---- | :---- | :----- |
    | 1 | sw_yin4 | 0000123.png |
    | 2 | sw_yin4 | 0000296.png |
    | ... | ... | ... |

### 讨论
1. $\color{red}{多人标注的一些约定}$
    - 目的：数据集是一个整体，保证**标注一致性**
    - 需不需要裁剪出飞鸟存在的区域，放大该区域，保存这样的图片？
        - 实际运行的时候并不太可能裁剪出每个区域，再做检测
    * 图片上的鸟通常很小，我们的标注框，是紧密耦合飞鸟目标，还是稍稍大一些？

4. 目前计划是以图片形式发布数据集，是否考虑其他形式，以视频形式发布可行吗？
    - 标注怎么解决？算法标完，无法人工修正，还是得用图片

1. 制作飞鸟数据集是一个开放性的问题，除了标注，还需要做哪些事情，才能形成一个好的论文？
    - 分析哪些特性：参考tinyperson，像素框大小统计、局部区域放大
    - 和哪些数据集作对比：划分训练集、测试集，测试集不提供标签，后期还需做评估服务器？
    - 和哪些方法做对比，提供基线模型
    - 在其他数据集做实验，使用相同模型，如果本数据集效果不好，表明挑战非常大

2. 目前有哪些优势、问题？
    * 优势
        - 机场布置摄像头，获取真实数据，门槛高
        - 带有**时间序列性质**的图片数据集，不同于零散拼凑而成的图
        - 问题很特殊，有实际场景，机场飞鸟检测和三维定位$\rightarrow$智能驱鸟
    * 问题
        - 类别单调：因为像素太小，无法给飞鸟细分类：天鹅、老鹰、麻雀...
        - 图片密度大：有鸟的场景，每帧图像都保存（1秒约30帧），不够均匀和丰富

2. 建立发布数据集的网站，目前可行的方案是 Git Pages，大家有什么建议？
    - 能够公开访问，提供介绍和下载链接
    - 只发布内容，不需要和用户交互
    - 简单美观的界面
