---
layout: post
title: OpenCV Video Analysis
tags: [Computer Vision, OpenCV]
gh-repo: opencv/opencv
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

### 视频处理
##### 1. 提取前景模版--[当前帧和背景模型之间执行减法运算]

```python
    from __future__ import print_function
    import cv2 as cv
    import argparse
    # argparse将命令行全部信息解析成python数据类型可使用的数据
    parser = argparse.ArgumentParser(description='此程序显示如何使用\提供的背景减法OpenCV。您可以同时处理视频和图像。') #创建一个解析器
    parser.add_argument('--input', type=str, help='视频/图像序列路径', default='vtest.avi')
    parser.add_argument('--algo', type=str, help='背景减法方法(KNN, MOG2).', default='MOG2')
    # parser.add_argument(命名/列表，类型，...) 添加参数
    args = parser.parse_args() #解析参数

    # 创建背景减法器对象
    if args.algo == 'MOG2':
        # cv.createBackgroundSubtractorMOG2(history长度,像素与模型之间的Mahalanobis距离平方的阈值,是否检测并标记阴影)
        backSub = cv.createBackgroundSubtractorMOG2()
    else:
        backSub = cv.createBackgroundSubtractorKNN() #参数同上
        
    # 读取输入视频或输入图像序列
    capture = cv.VideoCapture(cv.samples.findFileOrKeep(args.input))
    if not capture.isOpened:
        print('Unable to open: ' + args.input)
        exit(0)
    while True:
        ret, frame = capture.read() #获取每一帧
        if frame is None:
            break
        
        fgMask = backSub.apply(frame) #将背景应用在每一帧，用于计算前景蒙版和更新背景
        
        cv.rectangle(frame, (10, 2), (100,20), (255,255,255), -1) #frame上绘制白色矩形，用于突出显示黑色的帧编号
        # 在图像上绘制文字cv.putText(待绘制的图像，待绘制文字，文本框左下角，字体，字体尺寸，线条颜色，线条宽度)
        cv.putText(frame, str(capture.get(cv.CAP_PROP_POS_FRAMES)), (15, 15),
                cv.FONT_HERSHEY_SIMPLEX, 0.5 , (0,0,0))  #将当前帧号标注在左上角
        
        # 显示当前帧和前景蒙版
        cv.imshow('Frame', frame)
        cv.imshow('FG Mask', fgMask)
        
        keyboard = cv.waitKey(30)
        if keyboard == 'q' or keyboard == 27:
            break
```

##### 2. 跟踪视频对象--Meanshift和Camshift
* 原理：我们通常会传递直方图反投影图像和初始目标位置。当对象移动时，显然该移动会反映在直方图反投影图像中。结果，meanshift算法将我们的窗口以最大密度移动到新位置。

###### Meanshift

```python
    import numpy as np
    import cv2 as cv
    import argparse
    parser = argparse.ArgumentParser(description='This sample demonstrates the meanshift algorithm. \
                                                The example file can be downloaded from: \
                                                https://www.bogotobogo.com/python/OpenCV_Python/images/mean_shift_tracking/slow_traffic_small.mp4')
    parser.add_argument('image', type=str, help='图像文件路径')
    args = parser.parse_args()

    cap = cv.VideoCapture(args.image)
    # 读视频的第一帧
    ret,frame = cap.read()
    # 设置窗口的初始位置
    x, y, w, h = 300, 200, 100, 50 
    track_window = (x, y, w, h)
    # 设置用于跟踪的ROI
    roi = frame[y:y+h, x:x+w] #取一帧的某块(x,y,w,h)
    hsv_roi =  cv.cvtColor(roi, cv.COLOR_BGR2HSV) #颜色从BGR变成HSV
    # 取出在上下限范围内的图像区域 cv.inRange(目标图像，阈值下限，阈值上限)
    mask = cv.inRange(hsv_roi, np.array((0., 60.,32.)), np.array((180.,255.,255.)))

    roi_hist = cv.calcHist([hsv_roi],[0],mask,[180],[0,180]) #获取hsv_roi的直方图
    cv.normalize(roi_hist,roi_hist,0,255,cv.NORM_MINMAX) #归一化到(0,255)内
    # 设置终止条件，可以是10次迭代或至少移动1 pt
    term_crit = ( cv.TERM_CRITERIA_EPS | cv.TERM_CRITERIA_COUNT, 10, 1 )
    while(1):
        ret, frame = cap.read()
        if ret == True:
            hsv = cv.cvtColor(frame, cv.COLOR_BGR2HSV)
            dst = cv.calcBackProject([hsv],[0],roi_hist,[0,180],1) #获取输入图像直方图的反向投影图
            # 应用meanshift获取新位置
            # cv.meanShift(反向投影图，跟踪目标初始矩形框，算法结束条件)
            ret, track_window = cv.meanShift(dst, track_window, term_crit)
            # 在图像上绘制
            x,y,w,h = track_window
            img2 = cv.rectangle(frame, (x,y), (x+w,y+h), 255,2)
            cv.imshow('img2',img2)
            k = cv.waitKey(30) & 0xff
            if k == 27:
                break
        else:
            break
```

###### Camshift

```python
    import numpy as np
    import cv2 as cv
    import argparse
    parser = argparse.ArgumentParser(description='This sample demonstrates the camshift algorithm. \
                                                The example file can be downloaded from: \
                                                https://www.bogotobogo.com/python/OpenCV_Python/images/mean_shift_tracking/slow_traffic_small.mp4')
    parser.add_argument('image', type=str, help='path to image file')
    args = parser.parse_args()
    cap = cv.VideoCapture(args.image)
    ret,frame = cap.read()
    x, y, w, h = 300, 200, 100, 50 
    track_window = (x, y, w, h)
    roi = frame[y:y+h, x:x+w]
    hsv_roi =  cv.cvtColor(roi, cv.COLOR_BGR2HSV)
    mask = cv.inRange(hsv_roi, np.array((0., 60.,32.)), np.array((180.,255.,255.)))
    roi_hist = cv.calcHist([hsv_roi],[0],mask,[180],[0,180])
    cv.normalize(roi_hist,roi_hist,0,255,cv.NORM_MINMAX)
    term_crit = ( cv.TERM_CRITERIA_EPS | cv.TERM_CRITERIA_COUNT, 10, 1 )
    while(1):
        ret, frame = cap.read()
        if ret == True:
            hsv = cv.cvtColor(frame, cv.COLOR_BGR2HSV)
            dst = cv.calcBackProject([hsv],[0],roi_hist,[0,180],1)
            # 应用Camshift获取新位置
            # cv.CamShift()参数同上
            ret, track_window = cv.CamShift(dst, track_window, term_crit)
            # 在图像上绘制
            pts = cv.boxPoints(ret)
            pts = np.int0(pts)
            img2 = cv.polylines(frame,[pts],True, 255,2)
            cv.imshow('img2',img2)
            k = cv.waitKey(30) & 0xff
            if k == 27:
                break
        else:
            break
```



##### 3. 光流

###### 跟踪计算稀疏特征集的光流

```python
    import numpy as np
    import cv2 as cv
    import argparse
    parser = argparse.ArgumentParser(description='This sample demonstrates Lucas-Kanade Optical Flow calculation. \
                                                The example file can be downloaded from: \
                                                https://www.bogotobogo.com/python/OpenCV_Python/images/mean_shift_tracking/slow_traffic_small.mp4')
    parser.add_argument('image', type=str, help='path to image file')
    args = parser.parse_args()
    cap = cv.VideoCapture(args.image)
    # 用于shiTomasi角点检测的参数[maxCorners角点最大数量、quanlityLevel可接受最小特征值、minDistance角点间最小距离、blockSize计算角点时参与运算的区域大小]
    feature_params = dict( maxCorners = 100,
                        qualityLevel = 0.3,
                        minDistance = 7,
                        blockSize = 7 )
    # lucas kanade光流的参数[winSize每个金字塔等级的搜索窗口的winSize大小、maxLevel最大金字塔等级数、criteria指定迭代搜索算法的终止条件]
    lk_params = dict( winSize  = (15,15),
                    maxLevel = 2,
                    criteria = (cv.TERM_CRITERIA_EPS | cv.TERM_CRITERIA_COUNT, 10, 0.03))
    # 创建一些随机颜色
    color = np.random.randint(0,255,(100,3))
    # 拍摄第一帧并在其中找到角点
    ret, old_frame = cap.read()
    old_gray = cv.cvtColor(old_frame, cv.COLOR_BGR2GRAY) #转为灰度图
    # 返回角点cv.goodFeaturesToTrack(输入图像，mask如果指定，它的维度必须和输入图像一致，且在mask值为0处不进行角点检测，feature_params其他参数)
    p0 = cv.goodFeaturesToTrack(old_gray, mask = None, **feature_params)
    # 创建用于绘画的遮罩图像[全0数组]
    mask = np.zeros_like(old_frame)
    while(1):
        ret,frame = cap.read()
        frame_gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
        # 计算光流量
        # 使用带有金字塔的迭代Lucas-Kanade方法计算稀疏特征集的光流 cv.calcOpticalFlowPyrLK(第一帧图像、下一帧图像、...、计算的光流点、金字塔缩放比例、初始图像金字塔层数、lk_params)
        p1, st, err = cv.calcOpticalFlowPyrLK(old_gray, frame_gray, p0, None, **lk_params)
        # 选择好点
        good_new = p1[st==1] 
        good_old = p0[st==1]
        # 绘制轨道
        for i,(new,old) in enumerate(zip(good_new, good_old)):
            a,b = new.ravel()  #转为一维数组，(x,y)
            c,d = old.ravel()
            mask = cv.line(mask, (a,b),(c,d), color[i].tolist(), 2)
            frame = cv.circle(frame,(a,b),5,color[i].tolist(),-1)
        img = cv.add(frame,mask)
        cv.imshow('frame',img)
        k = cv.waitKey(30) & 0xff
        if k == 27:
            break
        # 现在更新前一帧和前一个
        old_gray = frame_gray.copy()
        p0 = good_new.reshape(-1,1,2)
```

###### 查找密集光流

```python
import numpy as np
    import cv2 as cv
    cap = cv.VideoCapture(cv.samples.findFile("vtest.avi"))
    ret, frame1 = cap.read()
    prvs = cv.cvtColor(frame1,cv.COLOR_BGR2GRAY)
    hsv = np.zeros_like(frame1)
    hsv[...,1] = 255
    while(1):
        ret, frame2 = cap.read()
        next = cv.cvtColor(frame2,cv.COLOR_BGR2GRAY)
        # cv.calcOpticalFlowFarneback(前一帧、后一帧、金字塔缩放比例、金字塔层数、窗口大小、金字塔每层迭代次数、在每个像素点计算多项式展开的相邻像素点个数、标准差[与前一个参数有对应关系]、flag[OPTFLOW_USE_INITIAL_FLOW/OPTFLOW_FARNEBACK_GAUSSIAN])
        # 返回一个两通道的光流向量，实际上是每个点的像素位移值
        flow = cv.calcOpticalFlowFarneback(prvs,next, None, 0.5, 3, 15, 3, 5, 1.2, 0)
        # cv.cartToPolar()笛卡尔坐标转化为极坐标
        mag, ang = cv.cartToPolar(flow[...,0], flow[...,1]) 
        hsv[...,0] = ang*180/np.pi/2 #角度
        hsv[...,2] = cv.normalize(mag,None,0,255,cv.NORM_MINMAX) #归一化到0-255范围之内
        bgr = cv.cvtColor(hsv,cv.COLOR_HSV2BGR) #再从HSV转换成BGR
        cv.imshow('frame2',bgr)
        k = cv.waitKey(30) & 0xff
        if k == 27:
            break
        elif k == ord('s'):
            cv.imwrite('opticalfb.png',frame2)
            cv.imwrite('opticalhsv.png',bgr)
        prvs = next
```
