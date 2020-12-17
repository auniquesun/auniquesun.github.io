---
layout: post
title: OpenCV Core Operation
tags: [Computer Vision, OpenCV]
gh-repo: opencv/opencv
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

### Table of Contents
1. Basic Operations on Images  
2. Arithmetic Operations on Images  
3. Performance Measurement and Improvement Techniques  

{: .box-note}
图片基础操作

{% highlight python linenos %}
    """
    一个很重要的认知：读取出来的图片，是numpy数组
                因此，numpy对数组的操作，都可以用上
    """
    img = cv.imread('messi5.jpg')
    """
    x,y 确定一个像素点：这个像素点是3维np向量，分别代表 b/g/r 值
    """
    px = img[100,100]
    print( px )

    blue = img[100,100,0]
    print( blue )

    img[100,100] = [255,255,255]
    print( img[100,100] )

    """
    像上面那样，单独访问每个像素值很慢，没利用好numpy
    使用numpy中的 array.item() 和 array.itemset() 是更好的方式
    """
    img.item(10,10,2)
    # 最后的参数100，是修改后的值
    img.itemset((10,10,2),100)
    img.item(10,10,2)

    # 图片维度
    img.shape
    # 图片像素总数
    img.size

    """
    访问或修改图片的一块区域 —— region of interest
    """
    ball = img[280:340, 330:390]
    img[273:333, 100:160] = ball

    """
    分割/合并图像的各个通道
    """
    b, g, r = cv.split(img)
    cv.merge(b, g, r)
    # 把蓝色通道全置0
    img[:,:,0] = 0
    # 得到红色通道的值
    r = img[:, :, 2]

    # 操作图像边缘
    """
    cv.copyMakeBorder()，各个参数
        1：np图像
        2/3/4/5：选定的 top/bottom/left/right 区域边界
        6：边界操作类型
        7：边界色彩值，包含三个值的list，当边界类型为BORDER_CONSTANT时设置
    """
    BLUE = [255,0,0]
    img1 = cv.imread('opencv-logo.png')

    replicate = cv.copyMakeBorder(img1,10,10,10,10,cv.BORDER_REPLICATE)
    reflect = cv.copyMakeBorder(img1,10,10,10,10,cv.BORDER_REFLECT)
    reflect101 = cv.copyMakeBorder(img1,10,10,10,10,cv.BORDER_REFLECT_101)
    wrap = cv.copyMakeBorder(img1,10,10,10,10,cv.BORDER_WRAP)
    constant= cv.copyMakeBorder(img1,10,10,10,10,cv.BORDER_CONSTANT,value=BLUE)
    plt.subplot(231),plt.imshow(img1,'gray'),plt.title('ORIGINAL')
    plt.subplot(232),plt.imshow(replicate,'gray'),plt.title('REPLICATE')
    plt.subplot(233),plt.imshow(reflect,'gray'),plt.title('REFLECT')
    plt.subplot(234),plt.imshow(reflect101,'gray'),plt.title('REFLECT_101')
    plt.subplot(235),plt.imshow(wrap,'gray'),plt.title('WRAP')
    plt.subplot(236),plt.imshow(constant,'gray'),plt.title('CONSTANT')
{% endhighlight %}

{: .box-note}
图片像素位运算

{% highlight python linenos %}
    # 图像加权融合
    img1 = cv.imread('ml.png')
    img2 = cv.imread('opencv-logo.png')
    dst = cv.addWeighted(img1,0.7,img2,0.3,0)
    cv.imshow('dst',dst)
    cv.waitKey(0)
    cv.destroyAllWindows()

    """
    只把opencv logo叠加到另一幅图片
    """
    img1 = cv.imread('messi5.jpg')
    img2 = cv.imread('opencv-logo-white.png')
    # I want to put logo on top-left corner, So I create a ROI
    rows,cols,channels = img2.shape
    roi = img1[0:rows, 0:cols]
    # Now create a mask of logo and create its inverse mask also
    img2gray = cv.cvtColor(img2,cv.COLOR_BGR2GRAY)
    ret, mask = cv.threshold(img2gray, 10, 255, cv.THRESH_BINARY)
    mask_inv = cv.bitwise_not(mask)
    # Now black-out the area of logo in ROI
    img1_bg = cv.bitwise_and(roi,roi,mask = mask_inv)
    # Take only region of logo from logo image.
    img2_fg = cv.bitwise_and(img2,img2,mask = mask)
    # Put logo in ROI and modify the main image
    dst = cv.add(img1_bg,img2_fg)
    img1[0:rows, 0:cols ] = dst
    cv.imshow('res',img1)
    cv.waitKey(0)
    cv.destroyAllWindows()
{% endhighlight %}

{: .box-note}
OpenCV 计算图片操作运行时间 —— 代码优化

{% highlight python linenos %}
    """
    记录一段代码的运行时间：
        cv.getTickCount()   获取clock-cycles次数
        cv.getTickFrequency()   获取每秒 clock-cycles次数

        cv.getTickCount() 之差，就是代码运行经历的clock-cycles总次数
                        再除以cv.getTickFrequency()就得到了运行时间
    """
    e1 = cv.getTickCount()
    # your code execution
    e2 = cv.getTickCount()
    time = (e2 - e1)/ cv.getTickFrequency()

    img1 = cv.imread('messi5.jpg')
    e1 = cv.getTickCount()
    for i in range(5,49,2):
        img1 = cv.medianBlur(img1,i)
    e2 = cv.getTickCount()
    t = (e2 - e1)/cv.getTickFrequency()
    print( t )
    # Result I got is 0.521107655 seconds

    """
    许多情况下，OpenCV代码的时空复杂度都比较好，但不会是全部
        视觉SLAM十四讲中，一些手写的代码就比OpenCV运行速度快
        下面的代码是在 IPython 运行的
    """
    # check if optimization is enabled (default is enabled)
    In [5]: cv.useOptimized()
    Out[5]: True
    In [6]: %timeit res = cv.medianBlur(img,49)
    10 loops, best of 3: 34.9 ms per loop
    # Disable it
    In [7]: cv.setUseOptimized(False)
    In [8]: cv.useOptimized()
    Out[8]: False
    In [9]: %timeit res = cv.medianBlur(img,49)
    10 loops, best of 3: 64.1 ms per loop

    """
    %timeit 能直接测单行代码运行时间

    opencv比numpy快很多
    """
    In [10]: x = 5
    In [11]: %timeit y=x**2
    10000000 loops, best of 3: 73 ns per loop
    In [12]: %timeit y=x*x
    10000000 loops, best of 3: 58.3 ns per loop
    In [15]: z = np.uint8([5])
    In [17]: %timeit y=z*z
    1000000 loops, best of 3: 1.25 us per loop
    In [19]: %timeit y=np.square(z)
    1000000 loops, best of 3: 1.16 us per loop

    # opencv比numpy快25倍
    In [35]: %timeit z = cv.countNonZero(img)
    100000 loops, best of 3: 15.8 us per loop
    In [36]: %timeit z = np.count_nonzero(img)
    1000 loops, best of 3: 370 us per loop
{% endhighlight %}