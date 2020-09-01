---
layout: post
title: Lab Server User Tutorials
gh-repo: ohmyzsh/ohmyzsh
gh-badge: [star, fork, follow]
tags: [Ubuntu, Shell, Data Sharing]
mathjax: true
comments: true
---

上一篇[post](https://auniquesun.com/2020-06-26-basic-developing-environments-for-vision-project-and-research/)从全局角度讲解了做视觉开发、研究的服务器基础环境配置问题，本文是对上篇文章的深入，讲解实施细节。具体来说，当 Ubuntu 系统装好以后，
* 如何为新用户创建友好的、易用的Shell —— 用户在Ubuntu服务器上操作，Shell是用户和系统最主要的交互工具
    * 有人可能好奇，Shell都是命令行操作，还有友好、易用之分？根据[新用户指南](#新用户指南)配置完成，你就切身体会到了：**原生的Shell只是让你能够使用系统，并不能让你高效方便的使用系统**
* 在多用户使用的情况下，如何共享数据资源 —— 节省时间、提高效率，比如常用的miniconda3安装包，下载一份多人使用即可
    * 服务器数据共享的方式有很多种，我们采用**文件夹权限共享**的方式

### 写在前面

{: .box-error}
**请大家严格按照本文要求操作** 

* 实验室服务器是一台**硬件配置较高**的机器，常用软件已经下载或配置好，并由专门的人员运维，满足不同项目需求和环境配置，为同学们科研和开发提供便利。

* 运维服务器的人员可能牺牲了他的休息时间、写论文时间、思考问题时间无偿为同学们提供服务，请大家遵循 “**便利自己也便利他人**” 原则，严格按照本文要求操作：
    * 请阅读[数据共享](#数据共享)部分 —— 用户主目录下不要存放大文件，若主目录占用磁盘空间超过15G，会被定期清除
    * 若使用 `Python` 开发，详见 [新用户指南](#新用户指南)第`7`条 安装 `miniconda`，**杜绝使用系统自带** `Python`  

* 欢迎你~ 祝你机器用得顺、代码写得好，论文发得早！


### 新用户指南
> 管理员创建好一个用户，系统会给用户配置默认的Shell —— `bash`，它对用户并不很友好；用户需要切换到对 **ta** 更好的 `zsh`

0. 【忽略这一步，已由管理员完成】首先需要安装 `zsh`，因为它没有内置在 Ubuntu 系统；确保有管理员权限，然后执行如下命令
    ```shell
    sudo apt install zsh
    ```

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

5. `Shell`里几乎都是用命令操作，命令补全功能为用户提供了便利。`zsh-autosuggestions`是一个命令补全plugin，安装方法如下
    * step 1：下载 `zsh-autosuggestions` 到指定目录
    ```shell
    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
    ```
    * step 2：打开文件 `~/.zshrc`，把 `zsh-autosuggestions` 添加到 plugins 列表
    ```shell
    plugins=(zsh-autosuggestions)
    ```

5. **为了使用GPU**，需要设置`cuda`环境变量，在`~/.zshrc`文件末尾添加如下两行代码
    ```shell
    export CUDA_HOME=/usr/local/cuda
    export PATH=$CUDA_HOME/bin:$PATH
    ```
    
6. 至此，默认的 `bash` 已切换成 `zsh`；以后配置环境变量，在 `~/.zshrc` 配置即可；修改后让变更生效，执行如下命令
    ```shell
    source ~/.zshrc
    ```

7. 若使用 `Python` 开发，请安装 `miniconda` 环境（不推荐安装`Anaconda`：`miniconda`即轻量又能实现同样的功能）
    * step 1：切换到共享软件目录
    ```shell
    cd /mnt/sdb/public/software
    ```
    * step 2：安装 `miniconda` 环境 —— 配置均为默认
    ```shell
    sh Miniconda3-latest-Linux-x86_64.sh
    ```

### 数据和软件共享
0. `/mnt/sdb` 目录挂载的是一块 4TB 的机械硬盘，磁盘空间较为充足；其子目录 `public/` 用于服务器多用户**_共享数据、存放大文件_**
* **注意：** 每个用户主目录下不要存放大的数据文件，大的数据文件存放于 `/mnt/sdb/public/data/<username>` 下
* 如果不清楚如何挂载机械硬盘到 `Ubuntu` 系统，参考这篇 [blog](https://medium.com/@sh.tsang/partitioning-formatting-and-mounting-a-hard-drive-in-linux-ubuntu-18-04-324b7634d1e0)；**用户不需要做这一步**，已由管理员完成


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
* 修改刚才建立的文件夹的权限，共享给其他用户
   ```shell
    chmod o+rx /mnt/sdb/public/data/hs
    ```

4. 用户将文件拷贝到 `/mnt/sdb/public/data/`、`/mnt/sdb/public/software/` 或者 `/mnt/sdb/public/data/<username>/`，**务必修改文件权限**，**_这样其他用户才能使用_**，修改命令如下
    ```shell
    chmod o+rx <filename>
    ```
    > e.g. chmod o+rx Miniconda3-latest-Linux-x86_64.sh

3. 用户对共享目录有读、写、执行权限，【**请勿删除、修改**】 `public/` 目录下的任何文件

### 相关文档
* 上一篇：[视觉项目研发环境配置](https://auniquesun.com/2020-06-26-basic-developing-environments-for-vision-project-and-research/)
* 下一篇：[服务器GPU配置](https://auniquesun.com/2020-07-13-GPU-utility-settings/)
