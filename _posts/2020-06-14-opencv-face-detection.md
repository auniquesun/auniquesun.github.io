---
layout: post
mathjax: true
comments: true
title:  "Face Detection with OpenCV in real time"
tags: [Face Detection, OpenCV]
gh-repo: shantnu/FaceDetect
gh-badge: [star, fork, follow]
---

最近做了一个 demo 演示程序，利用笔记本电脑的摄像头——webcam，实时追踪镜头中的人脸。做这个小项目是为了
搞通《机场飞鸟检测》项目的各个流程，包括摄像头输入、图像获取、物体识别、输出视频流。所以这篇文章就从这几
个方面介绍——用OpenCV实现实时人脸检测。


## 开发环境准备
0. Ubuntu 16.04，比较适合程序开发  
1. 安装 Python，这里我用的 miniconda3，方便配置针对具体项目的Python环境  
2. 安装 opencv-python，**不需要再下载 OpenCV 源码，自己再编译安装**，我之前一直以为要使用opencv-python，先要自己在系统手动编译安装 OpenCV 源码


## 获取摄像头输入
1. OpenCV 可以操作电脑摄像头，从摄像头读取一帧帧图片，这是它强大的地方，因为通常来说操作硬件会涉及到系统
调用，以及硬件驱动程序配置，OpenCV 都帮我们搞定了。

2. 调用摄像头，代码如下。VideoCapture这个函数可以读取**视频文件**、**摄像头**、**图片序列**，指定不同的参数即可；参数 0 表示从摄像头输入
    ```python
    import cv2

    video_capture = cv2.VideoCapture(0)
    ```


## 从摄像头获取图像
1. 读取摄像头数据后，是把它存在了内存区域，并不直接得到图片数据，还需经过一定转换，转换步骤如下
    ```python
    ret, frame = video_capture.read()
    ```
    - ret的类型是 bool，代表图片是否被正确读取；frame 就是一帧图片，有了图片就能做后续处理


## 人脸检测
1. 人脸检测技术，不管怎么变，目前输入的基本单元都是图片；现实视频场景中的人脸检测，也需要先获取到图片，和这个demo的道理一样

2. 人脸检测技术有很多，随着人脸图片库的丰富（像Facebook、商汤、旷世，几十亿张人脸图片库已经稀松平常），和深度学习技术的应用，人脸识别可以看作一个已经解决的问题。本文重在打通**物体实时追踪的各个环节，不过分追求检测效果**，
使用OpenCV自带的人脸检测算法，既方便又能达到demo展示的目的。

3. 输入图片，进行检测
    ```python
    # 这个xml文件是检测器的配置文件，需要提前下载好，网上有很多公开的
    cascPath = "haarcascade_frontalface_default.xml"
    faceCascade = cv2.CascadeClassifier(cascPath)

    # 转换成灰度图，个人认为是为了减少计算量
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    faces = faceCascade.detectMultiScale(
        gray,
        scaleFactor=1.2,
        minNeighbors=1,
        minSize=(40, 40)
    )

    # 在图片上画框，相当于改变frame某些像素颜色
    for (x, y, w, h) in faces:
        cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)
    ```
4. 上面只是处理了一帧图片，还达不到实时检测的效果，所以把这段代码**放在循环里**


## 输出视频流
1. 这看似一件很复杂的事情，需要把处理完的图片显示出来，才能看到效果。OpenCV让这变得简单起来，只需
    ```python
    cv2.imshow('Video', frame)
    ```
    - 播放图片序列，形成视频


## 本项目完整的代码
{% highlight python linenos %}
import cv2

cascPath = "haarcascade_frontalface_default.xml"
faceCascade = cv2.CascadeClassifier(cascPath)

video_capture = cv2.VideoCapture(0)

while True:
    ret, frame = video_capture.read()

    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    faces = faceCascade.detectMultiScale(
        gray,
        scaleFactor=1.2,
        minNeighbors=1,
        minSize=(40, 40)
    )

    for (x, y, w, h) in faces:
        cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)

    cv2.imshow('Video', frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

video_capture.release()
cv2.destroyAllWindows()
{% endhighlight %}


{: .box-note}
**Note:** 实现这样一个demo程序并不是困难的事情，费心费力的地方在于环境配置，也就是**开发环境准备**。如果从头开始做，真正
写程序调bug的时间不到1小时，但是开发环境准备需要一上午甚至一天。我想强调的是不论做软件开发还是算法研究，都需要有系统思维，因
为任何软件、算法都要运行在操作系统，**更具体说是操作系统上搭建的运行环境**，要配置稳定易用的系统环境，没有这一步什么牛逼的算法都跑不起来，更不要谈算法
效果。但是对于个体开发者而言，由于自身原因和软硬件配置等问题，配好一个项目的开发环境并不简单，不像科技公司早就为员工准备好了
友好的开发环境，所以平时除了关注算法本身，还要锻炼系统开发和配置能力。