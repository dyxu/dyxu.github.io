---
layout: post
title: Archlinux构建系统
author: pier
keywords: ["archlinux"]
description:
categories:
  - 技术
tags: ["linux"]

---

{% include JB/setup %}

## 1 pacman命令
--------------------------
在archlinux中通过pacman来管理软件，全称为`package manager`，使用方式如下：

1. 更新系统
 - `pacman -Syu` 更新整个系统
 - `pacman -Sy` 同步本地包数据库和远程的仓库
 - `pacman -Su` 根据本地包数据库更新系统
 - `pacman -Syu -ignore 包名` 更新系统时不升级制定包

2. 安装软件包
 - `pacman -S包名` 安装软件包，多个包间用空格分割
 - `pacman -Sy 包名` 先同步本地数据库，再安装软件包
 - `pacman -Sf 包名` 强制安装包
 - `pacman -Sv 包名` 安装包，并打印详细信息
 - `pacman -U 包名.pkg.tar.gz` 安装本地软件包

3. 删除软件包
 - `pacman -R 包名` 只删除制定包，不清理依赖包
 - `pacman -Rs 包名` 删除包并删除依赖项
 - `pacman -Rd 包名` 删除包时不检查其依赖
 - `pacman -Rn 包名` 删除包及其配置文件
 - `pacman -Rsn 包名` 删除包、依赖项及其配置文件

4. 查询同步包数据库
 - `pacman -Ss 关键字` 在远程包数据库中查询含有关键字的包
 - `pacman -Si 包名` 查询远程包数据库中该包的详细信息

5. 查询本地包数据库
 - `pacman -Q` 列出系统中已安装包的包名
 - `pacman -Qs 关键字`　在本地查询含有关键字的包
 - `pacman -Qi 包名` 在本地包数据库中查询有关包的详细信息
 - `pacman -Ql 包名` 查询并列出该包所含文件
 - `pacman -Qo /path/file` 查询某文件属于何包
 - `pacman -Qdt` 查询系统中的孤立包
 - `pacman -Qet` 查询系统中的不被因依赖而安装的包

6. 其他用法
 - `pacman -Sw 包名` 只下载包，不安装
 - `pacman -Sc` 清理未安装的包文件（pacman 下载的包文件缓存于 /var/cache/pacman/pkg/ 目录）
 - `pacman -Scc` 清理所有的缓存文件
 - `pacman -Rs $(pacman -Qdtq)` 递归删除系统中存在的孤立包

## 2 pacman原理
-----------------------------
pacman的原理是在远程仓库中下载压缩包到本地，再运行`pacman -U 包名.pkg.tar.xz`安装。

在运行 `pacman -S 包名`时， pacman首先会从/var/lib/pacman/sync/文件夹里的几个数据库中搜索到nginx包对应的所有依赖包， 并递归搜索相应依赖包的所有依赖包，统计出安装 nginx必须要安装的所有包， 再从/var/lib/pacman/local/文件夹里查询这些包是否已经被安装，如果已被安装，则忽略安装此包及其依赖， 接着pacman就会从/etc/pacman.d/mirrorlist文件里选择一个包下载地址， 把这些包下载到/var/cache/pacman/pkg/文件夹里，安装包到系统里， 同时更新 /var/lib/pacman/local/文件夹，用来注册这些新安装的包， 到此安装过程结束。

## 3 打包
------------------

首先我们看看软件包的结构，例如redis-3.2.9-1-x86_64.pkg.tar.xz.

```shell
$ tar xvJf redis-3.2.9-1-x86_64.pkg.tar.xz
$ tree redis
redis
├── etc
│   ├── logrotate.d
│   │   └── redis
│   └── redis.conf
└── usr
    ├── bin
    │   ├── redis-benchmark
    │   ├── redis-check-aof
    │   ├── redis-check-rdb
    │   ├── redis-cli
    │   ├── redis-sentinel -> redis-server
    │   └── redis-server
    ├── lib
    │   └── systemd
    │       └── system
    │           └── redis.service
    └── share
        └── licenses
            └── redis
                └── LICENSE
```

从目录树里我们可以看到，这就是安装后redis的所有文件， 其中有二进制文件和配置文件(usr/和etc/目录)， 并且路径与操作系统中的目录树相对应。
那么redis-3.2.9-1-x86_64.pkg.tar.xz包是如何产生的呢？其实，它是由archlinux的维护人员事先打包构建好后放到mirrorlist供pacman命令下载。而构建这个包，主要是依靠makepkg命令并配置PKGBUILD脚步自动完成下载、解压、编译并打包成archlinux包的。

## 4 PKGBUILD
---------------------

为了构建一个archlinux包， 我们只需编写PKGBUILD脚本， 然后运行makepkg命令， 其通过PKGBUILD脚本中参数和指令来自动化下载、解压、编译和打包全过程。

PKGBUILD非常好写，通过shell语法定义一些makepkg命令需要的参数和指令。

我们来看看上一小节中PKGBUILD中定义的参数：

- pkgname：包名
- pkgver：包版本
- pkgrel：包的 release number
- pkgdesc：包的简要概述
- arch：包编译的体系结构
- url：包官方网站 url
- depends：包的依赖包
- makedepends：编译这个包时需要的依赖包
- source：包源码的下载地址
- md5sums：上面 source 参数对应项的 md5sum，用来检测下载包的正确性
- build：编译指令
- package：打包指令
- 更多参数请参考[archlinux wiki](https://wiki.archlinux.org/index.php/PKGBUILD_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))