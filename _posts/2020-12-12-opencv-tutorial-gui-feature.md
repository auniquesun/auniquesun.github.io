---
layout: post
title: OpenCV GUI Feature
tags: [Computer Vision, OpenCV]
gh-repo: opencv/opencv
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

### Getting Started with Images
{% highlight python linenos %}
    import cv2 as cv

    """
        第二个参数: int
        cv.IMREAD_COLOR : 1
        cv.IMREAD_GRAYSCALE : 0
        cv.IMREAD_UNCHANGED : -1 (Loads image as such including alpha channel)
    """
    img = cv.imread('disparity.png', cv.IMREAD_COLOR)

    cv.imshow('disparity', img)


    """
    cv.waitKey() 的参数是一个数字，代表等待时间，单位是毫秒
                如果传入的数字是0，将无限期等待，直到键盘收到按键
                还可以指定响应特定按键，如下所示
    返回值类型：int
    """
    k = cv.waitKey(0) & 0xFF

    # 按键等于 'q'
    if k == ord('q'):
        cv.imwrite('disparity_bck.png', img)

    # 传入 window name
    # cv.destroyWindow('disparity')
    # cv.destroyAllWindows()


    import numpy
    from matplotlib import pyplot as plt

    plt.imshow(img, cmap='gray', interpolation='bicubic')
    plt.xticks([]), plt.yticks([])  # to hide tick values on X and Y axis
    plt.show()
{% endhighlight %}

### Getting Started with Videos

{: .box-note}
从摄像头获取视频输入

{% highlight python linenos %}
    import numpy as np
    import cv2 as cv

    """
    cv.VideoCapture(0) 参数形式：
        (1) 数字: 输入设备编号；这里0代表第一个设备，
                  如果有第二个设备，传入1，以此类推；
                  都是实时获取视频流输入
        (2) 字符串：视频文件路径
    """
    cap = cv.VideoCapture(0)
    if not cap.isOpened():
        print("Cannot open camera")
        exit()
    while True:
        # Capture frame-by-frame
        ret, frame = cap.read()
        
        # ret type: bool
        # if frame is read correctly ret is True
        if not ret:
            print("Can't receive frame (stream end?). Exiting ...")
            break
        # Our operations on the frame come here
        gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)

        # Display the resulting frame
        cv.imshow('frame', gray)
        if cv.waitKey(1) == ord('q'):
            break
    # When everything done, release the capture
    cap.release()
    cv.destroyAllWindows()
{% endhighlight %}

{: .box-note}
从文件获得视频输入

{% highlight python linenos %}
    import numpy as np
    import cv2 as cv

    # 传入视频路径
    cap = cv.VideoCapture('vtest.avi')
    while cap.isOpened():
        ret, frame = cap.read()
        # if frame is read correctly ret is True
        if not ret:
            print("Can't receive frame (stream end?). Exiting ...")
            break

        # 把彩色图转成灰度图
        gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
        cv.imshow('frame', gray)
        if cv.waitKey(1) == ord('q'):
            break
    cap.release()
    cv.destroyAllWindows()
{% endhighlight %}

{: .box-note}
保存视频

{% highlight python linenos %}
    import numpy as np
    import cv2 as cv

    cap = cv.VideoCapture(0)
    """
    Define the codec
        有两种形式：
            cv.VideoWriter_fourcc(*'XVID')
            cv.VideoWriter_fourcc('X','V','I','D')
    """
    fourcc = cv.VideoWriter_fourcc(*'XVID')

    """
    create VideoWriter object
        cv.VideoWriter() 参数：
            输出视频文件名 - 'output.avi'：string
            视频编码格式   - 限定在几个不同关键字，linux系统常用 *'XVID'
            视频速度 - fps：int
            图片分辨率 - (width, height)：tuple
    """
    out = cv.VideoWriter('output.avi', fourcc, 20.0, (640,  480))
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            print("Can't receive frame (stream end?). Exiting ...")
            break
        # 图片翻转，第二个参数
        #   0：以图片x中轴作镜像对称
        #   1：以图片y中轴作镜像对称
        frame = cv.flip(frame, 0)
        # write the flipped frame
        out.write(frame)
        cv.imshow('frame', frame)
        if cv.waitKey(1) == ord('q'):
            break
    # Release everything if job is finished
    cap.release()
    out.release()
    cv.destroyAllWindows()
{% endhighlight %}

{: .box-note}
制作调色板

{% highlight python lineos %}
    import cv2 as cv
    import numpy as np

    # 回调函数
    def nothing(x):
        pass

    img = np.zeros((300,512,3), np.uint8)
    cv.namedWindow('platte')

    """
    cv.createTrackbar 的5个参数分别是：
        trackbar name
        window name
        default value
        maximum value
        callback function: 默认参数是 trackbar position，这里什么都不做
    """
    cv.createTrackbar('R', 'platte', 0, 255, nothing)
    cv.createTrackbar('G', 'platte', 0, 255, nothing)
    cv.createTrackbar('B', 'platte', 0, 255, nothing)

    switch = '0: OFF \n1: ON'
    cv.createTrackbar(switch, 'platte', 0, 1, nothing)

    while(1):
        cv.imshow('platte', img)
        k = cv.waitKey(5) & 0xFF
        if k == ord('q'):
            break

        r = cv.getTrackbarPos('R', 'platte')
        g = cv.getTrackbarPos('G', 'platte')
        b = cv.getTrackbarPos('B', 'platte')
        s = cv.getTrackbarPos(switch, 'platte')

        if s==0:
            img[:] = 0
        else:
            img[:] = [r,g,b]

    cv.destroyWindow('platte')
{% endhighlight %}
