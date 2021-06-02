---
layout: post
title: Part 2 - Airport Bird Dataset
tags: [Dataset, Object Detection, Computer Vision]
gh-repo: tzutalin/labelImg
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

上次每人处理完一部分视频后，还存在一些问题：
1. 同一月份，图片分辨率不一致，吴登智例外，分辨率都是1k
    - 王硕有的视频 1920x1080，有的视频 3840x2160
	- 邵羽有的视频 1920x1080，有的视频 3840x2160
	- 蔡旭东有的视频 1920x1080，有的视频 3840x2160

2. 伪标注图片很多，没有具体统计，随机打开几个文件夹，就能看到
    - 不担心有鸟的图片上的伪标注，因为这总是我们需要的图片
    - 担心没鸟图片上有伪标注，倘若这样的图很多

3. 统一标准：什么情况下标注，什么情况下不标注

### 解决办法
1. 分辨率缩小一半，标签中的各项坐标缩小一半
2. 就得人工过一些了，云彩、飞机，尤其是云彩的问题，夏天还挺多的