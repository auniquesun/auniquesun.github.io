---
layout: post
title: Some Measures to Secure a Linux Server
tags: [Linux Server, Secure Measures, Intrusion Prevention, Brute Force Hacking]
gh-repo: fail2ban/fail2ban
gh-badge: [star, fork, follow]
mathjax: true
comments: true
---

服务器安全稳定运行是一个重要的问题，但是经常被我们忽视，尤其是像校园实验室这样的环境，没有专职人员运维服务器，
更多是临时指派学生管理，而同学们更多关心如何使用服务器完成自己的实验和论文，
对服务器安全防御方面的知识了解较少，导致实验室多用户的服务器经常出现各种各样的问题，而解决办法往往是重装系统，重装各种软件，
重新建立用户等等，并不能找准问题进行解决，而过几天又会出现同样的问题，循环往复，大量时间浪费在上面。

这篇博客总结一些简单有效的方法，大部分经过本人实践，旨在让服务器**长期安全稳定地运行**，为用户工作提供便利。在这些方法的指导下，本人管理的服务器还没出现过问题

1. 严格的权限管理
    - 服务器出现这样那样的问题，很多时候因为权限管理出了问题：很多用户有`sudo`权限，或者直接使用root操作
    - 多用户的服务器，当用`sudo`修改一些系统配置时，都不知道其他用户之前做了什么操作，这样很可能引起混乱，查找问题时困难增大
    - 解决方法：
        1. 仅指派一个管理员，为不同人员创建单独账号，默认为普通用户
        2. 管理员在必要时用`sudo`操作，不要使用`root`
        3. 管理员根据实验室的研究工作，把系统常用的软件环境配置好，并维护一个软件列表向大家公布，省去为每个用户单独配置的时间

2. 禁用`root`通过ssh远程登录
    - 互联网上的攻击程序很多，有的程序专门暴力破解`root`账号(用户名已知)，一旦被破解，服务器就归攻击者所有了
    - 解决方法：服务器创建好其他管理员，应当尽快禁用`root`的ssh登录模式，这样攻击程序破解`root`账号这条路就走不通了
    ```shell
    sudo vim /etc/ssh/sshd_config
    ```
    > PermitRootLogin no  

    ```shell
    sudo systemctl restart sshd
    ```

3. 安装`fail2ban`，封禁多次登录失败的ip地址
    - 攻击程序可能对`非root`账号进行暴力破解
    - 解决方法：用`fail2ban`监控用户登录过程，通过修改`iptables`，封禁登录失败次数超过`maxretry`的ip地址
    ```shell
    sudo apt install fail2ban
    sudo vim /etc/fail2ban/jail.local
    ```
    > [DEFAULT]  
    > ignoreip=127.0.0.1  
    > bantime=43200  
    > findtime=1800  
    > [sshd]  
    > maxretry=4  

    ```shell
    sudo systemctl restart fail2ban
    ```

4. 安装`pwquaility`，提升用户密码设置强度
    - 管理员创建密码时为方便，经常设得很简单或者有固定模式可循，这样被破解的风险骤增
    - 解决方法：用`pwquaility`设定，创建新用户时密码需满足的要求
    ```shell
    sudo vim /etc/security/pwquality.conf
    ```
    > minlen = 12   # 密码最少12位  
    > minclass = 4  # 至少包含：数字、大写、小写、其他符号这四类字符

5. 对于已创建的用户，启用`公钥-私钥`对免密登录模式，而非手动输入密码
    - `公钥-私钥`非对称加密机制，是密码学中提出的一种安全可靠的加密模式，它们成对产生，`公钥`负责加密，对外公开，私钥负责解密，由用户保管
    - 解决方法：生成`公钥-私钥`对；公钥放到服务器指定目录，对外公开；私钥放到用户主机指定目录，自己保存  

    ```shell
    ssh-keygen  # 生成`公钥-私钥`对  
    ssh-copy-id -i id_rsa.pub user@hostname   # 公钥写入服务器~/.ssh/authorized_keys文件  

    sudo vim /etc/ssh/sshd_config  
    ```  

    > PasswordAuthentication no  
    > UsePAM no  

    ```shell
    sudo systemctl restart sshd
    ```

6. 修改ssh默认端口
    - ssh默认端口为22，攻击程序首先会扫描这个端口，
    - 解决方法：在条件允许的情况下修改为其他端口
    ```shell
    sudo vim /etc/ssh/sshd_config
    ```
    > Port 23456    # ssh指定为连接23456端口  

    ```shell
    sudo systemctl restart sshd
    ```

7. 禁用`ipv6`，在系统用不到该功能的情况下
    - `ipv6`是下一代网络地址模式，实验室很多机器其实用不到这个功能，默认开启的状态会带来很多安全风险
    - 解决方法：在系统层面，禁用`ipv6`能力
    ```shell
    sudo vim /etc/ssh/sshd_config
    ```
    > AddressFamily inet    # 设为inet，表示禁用ipv6  

    ```shell
    sudo systemctl restart sshd
    ```

8. 定期主动更新系统软件
    - 系统软件都在不断更新迭代：修补漏洞，增强功能。这需要用户主动更新软件，才能应用这些更新
    - 解决方法：设定一个时间周期，到了时间主动更新软件
    ```shell
    sudo apt update
    sudo apt upgrade
    ```