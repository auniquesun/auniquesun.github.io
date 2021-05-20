---
layout: post
title: Finetuning Disparity Parameters Using OpenCV GUI
tags: [Disparity Map, OpenCV, Computer Vision]
gh-repo: spmallick/learnopencv
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

### 双目视差图：可视化化参数调优

```python
import cv2 as cv


# callback function of trackbar
def nothing(x):
    pass


imgL_gray = cv.imread('data/im0.png', 0)
imgR_gray = cv.imread('data/im1.png', 0)

# Reading the mapping values for stereo image rectification
cv_file = cv.FileStorage("data/stereo_rectify_maps.xml", cv.FILE_STORAGE_READ)
Left_Stereo_Map_x = cv_file.getNode("Left_Stereo_Map_x").mat()
Left_Stereo_Map_y = cv_file.getNode("Left_Stereo_Map_y").mat()
Right_Stereo_Map_x = cv_file.getNode("Right_Stereo_Map_x").mat()
Right_Stereo_Map_y = cv_file.getNode("Right_Stereo_Map_y").mat()

# rectify stereo images
imgL_rectified = cv.remap(imgL_gray, Left_Stereo_Map_x, Left_Stereo_Map_y, ,cv.INTER_LANCZOS4, cv.BORDER_CONSTANT, 0)
imgR_rectified = cv.remap(imgR_gray, Right_Stereo_Map_x, Right_Stereo_Map_y, ,cv.INTER_LANCZOS4, cv.BORDER_CONSTANT, 0)

# create a window for trackbars
cv.namedWindow('disp',cv.WINDOW_NORMAL)
cv.resizeWindow('disp',600,600)

cv.createTrackbar('numDisparities', 'disp', 1, 17, nothing)
cv.createTrackbar('blockSize', 'disp', 5, 50, nothing)
cv.createTrackbar('preFilterType', 'disp', 1, 1, nothing)
cv.createTrackbar('preFilterSize', 'disp', 2, 25, nothing)
cv.createTrackbar('textureThreshold','disp',10,100,nothing)
cv.createTrackbar('uniquenessRatio','disp',15,100,nothing)
cv.createTrackbar('speckleRange','disp',0,100,nothing)
cv.createTrackbar('speckleWindowSize','disp',3,25,nothing)
cv.createTrackbar('disp12MaxDiff','disp',5,25,nothing)
cv.createTrackbar('minDisparity','disp',5,25,nothing)

# initiate an instance of stereo brute matching algorithm
stereo = cv.StereoBM_create()

while True:
    # get values of trackbars
    numDisparities = cv.getTrackbarPos('numDisparities', 'disp')*16
    blockSize = cv.getTrackbarPos('blockSize','disp')*2 + 5
    preFilterType = cv.getTrackbarPos('preFilterType','disp')
    preFilterSize = cv.getTrackbarPos('preFilterSize','disp')*2 + 5
    preFilterCap = cv.getTrackbarPos('preFilterCap','disp')
    textureThreshold = cv.getTrackbarPos('textureThreshold','disp')
    uniquenessRatio = cv.getTrackbarPos('uniquenessRatio','disp')
    speckleRange = cv.getTrackbarPos('speckleRange','disp')
    speckleWindowSize = cv.getTrackbarPos('speckleWindowSize','disp')*2
    disp12MaxDiff = cv.getTrackbarPos('disp12MaxDiff','disp')
    minDisparity = cv.getTrackbarPos('minDisparity','disp')

    # set params for stereo matching algorithm
    stereo.setNumDisparities(numDisparities)
    stereo.setBlockSize(blockSize)
    stereo.setPreFilterType(preFilterType)
    stereo.setPreFilterSize(preFilterSize)
    stereo.setPreFilterCap(preFilterCap)
    stereo.setTextureThreshold(textureThreshold)
    stereo.setUniquenessRatio(uniquenessRatio)
    stereo.setSpeckleRange(speckleRange)
    stereo.setSpeckleWindowSize(speckleWindowSize)
    stereo.setDisp12MaxDiff(disp12MaxDiff)
    stereo.setMinDisparity(minDisparity)

    disparity = stereo.compute(imgL, imgR)

    # convert to float32
    disparity = disparity.astype(np.float32)

    # scale down the disparity values and normalize them
    disparity = (disparity/16.0 - minDisparity)/numDisparities

    cv.imshow('disp', disparity)
    
    if cv.waitKey(0) == 27:
        print('Normal exit!!')
        break
```

遇到的问题：$\textcolor{red}{为什么Trackbar会占据那么长一段？为什么下面图片大小显示不完整呢？}$