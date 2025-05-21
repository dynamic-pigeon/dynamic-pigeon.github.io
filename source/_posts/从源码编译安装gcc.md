---
title: 从源码编译安装gcc
author: Dynamic_Pigeon
avatar: https://s3.bmp.ovh/imgs/2024/10/08/ca40e9bf152dbbf0.jpg
authorLink: dynamic-pigeon.github.io
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
categories: 技术
comments: true
date: 2025-01-27 18:42:23
updated: 2025-01-27 18:42:23
tags:
    - gcc
keywords:
    - gcc
description:
photos: https://s3.bmp.ovh/imgs/2025/01/27/6ee574e3a969b70c.png
---

这是一个个人编译安装 gcc 的存档。

本人操作系统 Debian-12

## 下载源码以及依赖

```text
http://gcc.gnu.org/install/ # 官方下载源
http://mirrors.nju.edu.cn/gnu/gcc/ # 南京大学源
https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/ # 清华源
```

可以选择自己需要的版本进行下载，这里我下载的是 `gcc-14.2`。

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/gcc-14.2.0/gcc-14.2.0.tar.xz
tar xf gcc-14.2.0.tar.xz
cd gcc-14.2.0
# 这一步可能需要给执行权限 sudo chmod a+x ./contrib/download_prerequisites
# 或者 bash ./contrib/download_prerequisites 
./contrib/download_prerequisites 
```
## 配置并编译

```bash
mkdir build
cd build
# 安装到默认位置的话不需要使用 --prefix，需要其他语言自行添加
../configure --enable-checking=release \
    --enable-languages=c,c++ \
    --disable-multilib  \
    --enable-bootstrap  \
    --prefix=<your-path>
```

执行完毕后或有成功的提示，如果报错，请根据提示操作。

```bash
# 这里看你 cpu 有多少核心，我是 13900 所以给了 24 核
# 过程中可能会提示两个文件没有执行权限，给权限就行了
# 只查看 warning 和 error 使用 make -j 24 > /dev/null
make -j 24
```

大概等待 1~2 小时后就会编译完（看电脑性能）。

```bash
make install
```

## 配置路径

使用默认路径的应该可以跳过

```bash
# 在 ~/.bashrc 中
# 替换 <your-path> 为你设置的路径
export PATH="<your-path>/gcc-14.2/bin:$PATH"
export LD_LIBRARY_PATH="<your-path>/gcc-14.2/lib64:<your-path>/gcc-14.2/lib:$LD_LIBRARY_PATH"
```

当然也可以直接换掉原本的 gcc 编译器。

```bash
# 替换之前最好备份一下本来的编译器
# 也可以直接 sudo apt remove g++ gcc
ln -sf <your-path>/bin/gcc /usr/bin/gcc
ln -sf <your-path>/bin/g++ /usr/bin/g++
```

现在查看一下版本，应该是 gcc-14.2 了。

然后你大概会发现 gdb 查看 stl 的功能爆炸了，这时候你可能还需要重新编译一下最新的 gdb（偷笑