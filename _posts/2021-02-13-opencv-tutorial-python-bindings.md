---
layout: post
title: OpenCV-Python Bindings
tags: [Computer Vision, OpenCV]
gh-repo: opencv/opencv
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

这一部分讲 `OpenCV-Python Bindings`，它指什么？OpenCV源码是用C++写的，再用Python开发一次当然可以，但是比较繁琐，更简便的想法是用Python调C++程序，可不可行？`OpenCV-Python Bindings`做出来之后，表明这是行之有效且富有创造力的想法，这给我们的启发是：用一种语言开发的项目，无需再次开发，就能提供另一种语言的编程接口。遇到类似问题时，这是一种可行的解决思路。

### 涉及模块
```
opencv/
       |-- modules/python/
                          |-- CMakeFiles.txt
                          |-- src2/
                                   |-- gen2.py
                                   |-- hdr_parser.py
                                   |-- cv2.cpp
```
* CMakeFiles.txt : 检测可以由C++扩展到Python的模块，抓取C++头文件  
* gen2.py : 主要的python bindings生成脚本，CMakeFiles.txt抓取的头文件作为输入  
* hdr_parser.py : 头文件解析脚本（但不能解析所有），生成函数、类等解析细节（如函数名、输入参数类型、输入参数、返回类型），以Python列表形式返回  
* cv2.cpp : 一些比较复杂，需要手动提供wrapper的函数放在这个文件

### 最终效果
> So now only thing left is the compilation of these wrapper files which gives us cv2 module. So when you call a function, say res = equalizeHist(img1,img2) in Python, you pass two numpy arrays and you expect another numpy array as the output. So **these numpy arrays are converted to cv::Mat and then calls the equalizeHist() function in C++**. Final result, **res will be converted back into a Numpy array**. So in short, almost all operations are done in C++ which **gives us almost same speed as that of C++**.

### 一些细节
* In OpenCV, all algorithms are implemented in C++. But these algorithms can be used from different languages like Python, Java etc. This is made possible by the bindings generators. These generators create a bridge between C++ and Python which enables users to call C++ functions from Python. To get a complete picture of what is happening in background, a good knowledge of Python/C API is required. A simple example on extending C++ functions to Python can be found in official Python documentation. So extending all functions in OpenCV to Python by writing their wrapper functions manually is a time-consuming task. So OpenCV does it in a more intelligent way. OpenCV generates these wrapper functions automatically from the C++ headers using some Python scripts which are located in modules/python/src2. We will look into what they do.

* First, modules/python/CMakeFiles.txt is a CMake script which checks the modules to be extended to Python. It will automatically check all the modules to be extended and grab their header files. These header files contain list of all classes, functions, constants etc. for that particular modules.

* Second, these header files are passed to a Python script, modules/python/src2/gen2.py. This is the Python bindings generator script. It calls another Python script modules/python/src2/hdr_parser.py. This is the header parser script. This header parser splits the complete header file into small Python lists. So these lists contain all details about a particular function, class etc. For example, a function will be parsed to get a list containing function name, return type, input arguments, argument types etc. Final list contains details of all the functions, structs, classes etc. in that header file.

* But header parser doesn't parse all the functions/classes in the header file. The developer has to specify which functions should be exported to Python. For that, there are certain macros added to the beginning of these declarations which enables the header parser to identify functions to be parsed. These macros are added by the developer who programs the particular function. In short, the developer decides which functions should be extended to Python and which are not. Details of those macros will be given in next session.

* So header parser returns a final big list of parsed functions. Our generator script (gen2.py) will create wrapper functions for all the functions/classes/enums/structs parsed by header parser (You can find these header files during compilation in the build/modules/python/ folder as pyopencv_generated_*.h files). But there may be some basic OpenCV datatypes like Mat, Vec4i, Size. They need to be extended manually. For example, a Mat type should be extended to Numpy array, Size should be extended to a tuple of two integers etc. Similarly, there may be some complex structs/classes/functions etc. which need to be extended manually. All such manual wrapper functions are placed in modules/python/src2/cv2.cpp.