---
layout: post
title: 简明shell教程
author: dyxu
keywords: 
description:
categories:
  - 技术
tags: ["shell"]

---
{% include JB/setup %}

## Shell配置

### Bash Shell启动配置文件

用户在登录shell时，会执行相应的配置文件，对shell进行配置和定制。主要分为三种类型

1. 用户登录主机时，loginshell先执行/etc/profile，接着依次检查用户主目录是否有.bash\_profile 或 .bash\_login 或 .profile
文件，若存在，则执行且只执行其中的一个。
2. 在登录后执行shell时，分为两种情况：
    1. 执行交付式的shell。例如打开终端，此时bash会以此执行/etc/bash.bashrc和主目录下.bashrc文件。
    2. 执行shell脚本。例如sh test.sh，此时bash会检测BASH_ENV变量的内容，如果该变量有定义，就执行所提供的配置文件的内容。

最后需要注意的是：shell注销后，其主目录下.bash_logout文件会被执行，类似析构函数的作用。

### 管理Shell配置文件

对于管理员，可以通过修改以下三个文件或目录统一管理shell的配置

1. /etc/profile 只要用户登录，都要执行
2. /etc/bash.bashrc 所有用户启动交互式shell时都要执行
3. /etc/skel/ 该目录中存放着.bash\_logout、.bashrc和.profile文件，当新用户被创建时，这三个文件就会被拷贝到其主目录下作为
默认配置文件

## Shell脚本语法



