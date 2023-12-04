---
layout: post
title: Rendering Multi-View Images for ShapeNet Core55 using MatLab
tags: [Multi-view Images, Mesh, Rendering, MatLab]
gh-repo: kanezaki/SHREC2017_track3
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

SHREC17 是评测三维物体检索方法性能的基准数据集，这个数据集由普林斯顿大学的研究者根据 ShapeNet Core55 中的三维物体制作而成。
基于多视图的三维物体检索是学术界研究热点之一，多视图反映了三维物体不同角度的观测信息，许多研究工作通过该处理这些多视图设计
三维物体检索方法。SHREC17 提供的数据格式是 mesh，没有现成的多视图数据。

问题在于现有相关工作并未公开它们使用的 SHREC17 数据，和其中的作者联系后很多情况下并未得到回复，同时从互联网上也未找到公开可访问的多视数据版本。
因此为了在这样的多视数据集评测方法效果，只能根据 SHREC17 原始数据从头渲染，这个过程会遇到较多困难和问题。

为了解决这一痛点，笔者决定把自身从头成功渲染 shrec17 多视图数据的过程记录下来，便于相关人员学习借鉴，同时我们也公开了自己渲染好的
SHREC17 多视数据集，供大家直接使用。
这个方法具有通用性，若要得到其他 mesh 的多视图，按照这个流程操作即可。

### ShapeNet Core55 多视数据集
1. 把最重要的放前面，渲染好的 ShapeNet Core55 多视数据集传到了[百度网盘](https://pan.baidu.com/s/1MVBAybo5UC9Qpg-J5Gyv6w?pwd=72jx)，提取码：72jx

### 环境配置
1. Windows 10 或者 Ubuntu 18.04
    - Windows 10 安装 MatLab 可能比较直观，都是图形界面
    - 本人在 Ubuntu 18.04 安装了 MatLab，这是一台服务器，算力资源比较丰富
2. MatLab R2019b
    - 渲染脚本在 MatLab 命令行运行

### 原始数据和渲染代码
1. 从 [SHREC](https://shapenet.cs.stanford.edu/shrec17/) 下载原始数据，它是一整个压缩文件，名为 `shrec17.zip`
    - 上述网站年久失修，在浏览器打开时可能被提示安全风险，点击类似**仍然访问**即可
    - 文件大小为 51G，下载时长取决于网速快慢，服务器后台挂着下载即可

2. 渲染代码
    - 下载代码，这里沿用了 RotationNet 采用的代码，其中核心渲染代码来自 MVCNN
        ```shell
        git clone https://github.com/kanezaki/SHREC2017_track3.git
        ```

### 渲染过程和注意事项
1. 开始渲染：准备好数据和代码即可开始工作
    1. 在 linux shell 中输入如下命令打开 MatLab，进入 matlab command shell (若为 Windows 系统，直接打开 MatLab 软件即可)
        ```shell
        matlab -nodisplay
        ```

    2. 在 matlab command shell 中，切换到之前下载的**渲染代码**目录
        ```shell
        cd SHREC2017_track3
        ```

    3. 依次执行如下命令，渲染 `shrec17` 不同的 `split` 和 `version`
        - split 有三种模式：`train`, `val`, `test`
        - version 有两种：`normal`, `perturbed`
        - 因此总共有6种组合，才能将所有需要的数据渲染完毕
        ```shell
        render_SHREC('train_normal'); 
        render_SHREC('val_normal'); 
        render_SHREC('test_normal'); 
        render_SHREC('train_perturbed'); 
        render_SHREC('val_perturbed'); 
        render_SHREC('test_perturbed'); 
        ```

2. 注意事项
    1. `shrec17` 共有 51,162 个mesh物体 (`.obj` 文件格式)，分布在55个类别，有的类别物体很多有的则很少，顺序执行上述命令时会非常缓慢
        - 可能卡在某个大的mesh文件等待数个小时
        - 可能卡在某个数量多的 mesh 类别很久，一到两天卡着不动都很常见，但会给人程序死机的错觉
        - 渲染程序或者 matlab 相关package不完善，中途会出现类似 ``java out of memory'' 的错误，要 kill 掉渲染程序重新运行
            - **好消息**是之前渲染好的 mesh 会保存，下次开始不需重做

    2. 因此，建议渲染前根据 `shrec17` 提供的数据标签将 mesh 划分成 55 个类别，每个类别建立单独文件夹存放对应 mesh，然后并行渲染多类 mesh，速度会加快很多
        - 划分 mesh 类别的脚本见 [mesh分类脚本](#mesh分类脚本)
        - 多类并行渲染示例，其中 `cls1`, `cls9` 表示不同的 mesh 类别，依次类推
            ```
            render_SHREC('train_normal_cls1'); 
            ...
            render_SHREC('train_normal_cls9'); 
            ```

    3. 即使多个终端并行渲染，总体仍然比较耗时，我完成整个过程花了 10 天左右
        - **不包括**环境配置，搞清楚代码运行过程，和解决各种报错的时间
        - 如有需要，建议直接使用笔者渲染好的多视数据集

### mesh分类脚本
{: .box-note}
1. 将 mesh 文件拷贝到它所属的类别目录，`shrec17` 共有55个类别
2. 切换到 `SHREC2017_track3` 目录运行下列脚本，脚本名为 `divide_mesh_category.py`
```shell
cd SHREC2017_track3
# train_normal 可替换为其他 <split>_<version>
python divide_mesh_category.py train_normal
```

    {% highlight bash linenos %}
    # filename: divide_mesh_category.py
    import os
    import sys
    import numpy as np
    import shutil

    data_dir = 'data'
    label_dir = 'TXT'
    # eg. train_normal
    version = sys.argv[1]
    mesh_dir = os.path.join(data_dir, version)
    num_classes = 55

    for clsId in range(num_classes):
        target_dir = os.path.join(data_dir, f'{version}_{clsId}')
        if not os.path.exists(target_dir):
            os.mkdir(target_dir)

        label_file = os.path.join(label_dir, f'{version}_{clsId}.txt')
        with open(label_file) as fin:
            lines = sorted([line.strip() for line in fin.readlines() if len(line.strip()) != 0]) # remove empty line
            lines = np.array(lines)[::20]   # take one every 20 lines

            meshes = []
            for line in lines:
                labels = line.split(' ')
                if len(labels) == 2:
                    basename = os.path.basename(labels[0])
                    mesh_filename = basename[:6]
                    meshes.append(mesh_filename)

            for mesh in meshes:
                src_file = os.path.join(mesh_dir, f'{mesh}.obj')
                shutil.copy(src_file, target_dir)

        print(f'>>> class {clsId} done')
    {% endhighlight %}
