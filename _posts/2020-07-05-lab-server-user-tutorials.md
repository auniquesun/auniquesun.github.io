---
layout: post
title: Lab Server User Tutorials
gh-repo: ohmyzsh/ohmyzsh
gh-badge: [star, fork, follow]
tags: [Ubuntu, Shell, Data Sharing]
mathjax: true
comments: true
---

[上一篇 post](https://auniquesun.com/2020-06-26-basic-developing-environments-for-vision-project-and-research/)从全局角度讲解了做视觉开发、研究的服务器基础环境配置问题，本文是对上篇文章的深入，讲解实施细节。具体来说，当 Ubuntu 系统装好以后，
* 如何为新用户创建友好的、易用的Shell —— 用户在Ubuntu服务器上操作，Shell是用户和系统最主要的交互工具
    * 有人可能好奇，Shell都是命令行操作，还有友好、易用之分？根据[新用户指南](#新用户指南)配置完成，你就切身体会到了：**原生的Shell只是让你能够使用系统，并不能让你高效方便的使用系统**
* 在多用户使用的情况下，如何共享数据资源 —— 节省时间、提高效率，比如常用的miniconda3安装包，下载一份多人使用即可
    * 服务器数据共享的方式有很多种，我们采用**文件夹权限共享**的方式

### 新用户指南
> 管理员创建好一个用户，系统会给用户配置默认的Shell —— `bash`，它对用户并不很友好；用户需要切换到对 **ta** 更好的 `zsh`

1. 把Shell切换成 `zsh`，执行如下命令
    ```shell
    chsh -s $(which zsh)
    ```

2. 进入目录 `/mnt/sdb/public/software`，执行如下命令
    ```shell
    cd /mnt/sdb/public/software
    ```

3. 用 `zsh` 安装 `ohmyzsh`，执行如下命令
    ```shell
    sh ohmyzsh-install.sh
    ```
    * `NOTE:` 安装过程中，需要确认的地方输入 `y`；默认安装在了用户主目录下的这个位置 `~/.oh-my-zsh`

4. 改变 ohmyzsh 主题元素，执行如下命令
    ```shell
    vim ~/.zshrc
    ```
    * 找到 `ZSH_THEME=` 所在行，替换 `robbyrussell` 为 `pygmalion`
    ![](../img/zsh.png)
    
5. 至此，默认的 `bash` 已切换成 `zsh`；以后配置环境变量，在 `~/.zshrc` 配置即可；修改后让变更生效，执行如下命令
    ```shell
    source ~/.zshrc
    ```

### 数据共享
0. `/mnt/sdb` 目录挂载的是一块 4TB 的机械硬盘，磁盘空间较为充足；其子目录 `public/` 用于服务器多用户**_共享数据、存放大文件_**
* **注意：** 每个用户主目录下不要存放大的数据文件，大的数据文件存放于 `/mnt/sdb/public/<username>` 下

1. 共享文件夹目录结构：
    > `public/`
    > -            |--- `data/`       : 共享常用数据集、大的数据文件
    > -            |--- `software/`   : 共享常用软件、安装包
    > -            |--- `操作指南.txt`

2. 事先确认要存放的文件是常用的、**_有必要共享_**的，再放入对应的文件夹

3. 对于**用户自己的大数据文件**
* 每个用户主目录下不要存放 _大数据文件_，因为“根目录/”只剩400G+空间，这部分要留给一些重要的程序和软件用
* 大的数据文件存放于 `/mnt/sdb/public/data/<username>/` 目录下，将 `<username>` 替换成自己的用户名，建立目录
    ```shell
    mkdir /mnt/sdb/public/data/hs
    ```

3. 用户对共享目录有读、写、执行权限，【**请勿删除、修改**】 `public/` 目录下的任何文件

