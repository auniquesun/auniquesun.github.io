---
layout: post
title: Recovering 3D Plane
tags: [Computer Vision, 3D Reconstruction, Plane Segmentation]
gh-repo: fuy34/planerecover
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

* 论文名称：[Recovering 3D Planes from a Single Image via](https://openaccess.thecvf.com/content_ECCV_2018/html/Fengting_Yang_Recovering_3D_Planes_ECCV_2018_paper.html)

* 论文作者：Fengting Yang and Zihan Zhou（Pennsylvania State University）

* 收录情况：ECCV 2018

### 简介

## 主要方法

### 获取标准平面标签存在的困难
* （1）图片中的平面区域边界往往很复杂，让人工标注会费很多时间，但是训练神经网络又需要大量带标签的数据。（2）如何从图片准确抽取各个平面的3D表示参数，仍然是一个没有解决的问题。

* 一种解决思路是：利用RGB-D数据集，把像素转化成带深度值的3D表面图，然后用 multi-model 拟合算法对3D表面上的点进行聚类，但这也不是一项容易的任务。
    - 主要的困难是：设置一个合适的阈值，来区分一个点是不是属于一个 model instance（可理解为一个物体），该阈值的设置与选取的算法无关（听起来有点不太合理，阈值的设定和算法相关是很正常的事情）

* 为了解决这样的困难，本文使用了 **SYNTHIA** 数据集，该数据集提供了大量用真实图片合成的城市场景及其对应的深度图。数据集通过渲染一个城市生成，该城市在Unity开游戏平台上开发，所以得到的深度地图是不带噪声的（noise-free）。为了从3D点云检测到平面，本文利用了多模态拟合方法——J-Linkage。该方法与 RANSAC 相似，是基于采样一致性的方法。
    - J-Linkage 一个关键的参数是 $\epsilon$，控制模型 hypothesis 和对应hypothesis数据点的最大距离
    - 下图是设置不同 J-Linkage 生成平面分割图的效果
    ![](../img/post/planerecover_fig2.png)
    
    - 作者用大段落举这个例子，是想说明这种方法得到的平面标注数据质量并不很高，再用来训练神经网络容易学到错误的信息，目的是下文引出自己的方法，文章写作技巧很高


### Plane Structure-Induced Loss
1. 上文阐述了获得可靠数据标签的挑战，这使得本文另辟蹊径，开发了另一方法进行3D平面恢复。有一个具体的问题：是否可以利用大规模的RGB-D / 3D 数据集，训练神经网络识别平面等几何结构，于此同时不需获取几何结构的ground truth标签？
    - 一个重要的想法是：如果能从图片恢复出3D平面，那么就能用这些平面（部分地）解释场景的几何性质 —— 通常用3D点云表示
    - $\{I_i, D_i\}_{i=1}^n$ n 个数据对 $\sim$ (RGB image, depth map)，相机内参K已知，对于数据集中所有图片都相同

    - 图片 $I_i$ 的像素 $\textbf{q} = [x, y, 1]^T$，它对应的3D点
        - $$ Q = D_i(\textbf{q}) \cdot K^{-1} \textbf{q} $$

    - 三维向量 $\textbf{n} \in \mathbb{R}$ 表示场景中的一个平面
        - 如果 $Q$ 位于该平面，则有 $\textbf{n}^T Q = 1$

2. 基于以上观测，假设图片 $I_i$ 有 $m$ 个平面，本文的目标是训练一个神经网络输出：
    1. 每个像素的概率图 $S_i$，$S_i(\textbf{q})$ 是个(m+1)维的向量，其中第$j$个元素 $S_i^j(\textbf{q})$ 表示像素 $\textbf{q}$ 属于第$j$个平面的概率
    2. 图片 $I_i$ 所有的平面参数 $$\Pi_i = \{\textbf{n}_i^j\}_{j=1}^{m}$$，最小化目标函数
        - $$ \mathcal{L} = \sum_{i=1}^n \sum_{j=1}^m \left(  \right) + \alpha \sum_{i=1}^n \mathcal{L}_{reg}(S_i) $$
        - $\mathcal{L}_{reg}(S_i)$ 是正则化项，阻止网络生成平凡解 —— $S_i^0(\cdot) \equiv 1$，这样会把所有像素划分成 non-planar，$\alpha$ 是平衡因子
        - 上式把 plane segmentation 和 plane parameter estimation 统一考虑，与用两种监督信息分别训练网络的方法不同

    3. 以上公式，$|(\textbf{n}_i^j)^TQ - 1|$ 衡量了图像$I_i$中第j个平面的3D点$Q$的偏差，又直到$Q$必在射线 $\lambda K^{-1} \textbf{q}$上，$\lambda$ 是$\textbf{q}$的深度值。若点$Q$又在第$j$个平面，则有
        - $$(\textbf{n}_i^j) \cdot K^{-1} \textbf{q} = 1 \Rightarrow \lambda = \frac{1}{(\textbf{n}_i^j) \cdot \lambda K^{-1} \textbf{q}}$$
        - 因此，$\lambda$ 可看作点$\textbf{q}$在平面$(\textbf{n}_i^j)$的深度值

    4. 基于以上推导，有如下等式成立
        - $$ \|(\textbf{n}_i^j)^TQ - 1\ | = |(\textbf{n}_i^j)^T D_i(\textbf{q}) \cdot K^{-1} \textbf{q} - 1|$$
        - 可以看到，上式计算 $D_i(\textbf{q})$ 和 $\lambda$ 的差异，惩罚差异项
        - 因此，本文把 3D plane recovery 看成一个深度估计问题

### Incorporating Semantics for Planar/Non-Planar Classification
1. 这部分主要考虑正则项 $\mathcal{L}_{reg}(S_i)$。出发点是希望预测的平面能尽可能反映场景的几何性质，因此 $\mathcal{L}_{reg}(S_i)$ 要鼓励 plane prediction——通过交叉熵损失函数
    - 令 $p_{plane}(\textbf{q}) = \sum_{j=1}^m S_i^j(\textbf{q})$
    - $$ \mathcal{L}_{reg}(S_i) = \sum_{\textbf{q}} -1 \cdot \log(p_{plane}(\textbf{q})) - 0 \cdot \log(1 - p_{plane}(\textbf{q}))$$

    - 损失函数的设计有效鼓励神经网络解释图像中的每个像素——使用预测平面模型
    - 实际情况中，一些物体比另外的更可能形成有意义的平面，比如建筑物的正面，于此同时行人、汽车被看作 non-planar
    - 进一步，能否把这样的高层语义信息考虑进来，提高 planar 和 non-planar 的分类能力

2. 基于上文介绍，提出了利用数据集中的语义标签的方法，以 SYNTHIA 为例
    - 该数据集提供了13类城市场景的像素集语义标签信息
    - 把这些类分成
        - planar = {building, fence, road, sidewalk, lane-marking}
        - non-planar = {sky, vegetation, pole, car, traffic signs, pedestrians, cyclists, miscellaneous}
        - 这种划分不固定，与问题和数据集相关
    
    - 令 $z(\textbf{q}) = 1$， 当 $\textbf{q}$ 属于 planar类别；否则 $z(\textbf{q}) = 0$，重写损失函数
        - $$ \mathcal{L}_{reg}(S_i) = \sum_{\textbf{q}} -z(\textbf{q}) \cdot \log(p_{plane}(\textbf{q})) - (1-z(\textbf{q})) \cdot \log(1 - p_{plane}(\textbf{q}))$$

    - 因此，就把经过人工标注的高层语义利用了起来，并和传统的geometric methods设定阈值区分 planar 和 non-planar 的方法区分开来

### Network Architecture