---
title: ffmpeg编译支持Nvidia加速
date: 2024-04-17 10:13:06
tags:
    - ffmpeg
    - nvidia
---

本文以 ubuntu 系统为例

# 安装 cuda 和显卡驱动

-   **安装 cuda**

```shell
sudo apt update
sudo apt install nvidia-cuda-toolkit

#头文件目录 /usr/include
#库文件目录 /usr/lib/x86_64-linux-gnu/
#可以使用sudo apt-file list nvidia-cuda-dev命令查看头文件和库文件安装目录
```

-   **安装显卡驱动**
    首先用`sudo apt search nvidia-driver`查找有哪些可用的驱动，选择备注为`NVIDIA driver metapackage`的驱动

```shell
sudo apt install nvidia-driver-550

# 安装完成之后需要重启，重启之后用nvidia-smi命令验证驱动是否安装完成
```

# 编译 ffmpeg

参考文档

> https://docs.nvidia.com/video-technologies/video-codec-sdk/12.0/ffmpeg-with-nvidia-gpu/index.html

```shell
# FFmpeg requires separate git repository nv-codec-headers for NV-accelerated ffmpeg build.

# Clone ffnvcodec
git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git

# Install ffnvcodec
cd nv-codec-headers && sudo make install && cd –

# Install necessary packages
sudo apt-get install build-essential yasm cmake libtool libc6 libc6-dev unzip wget libnuma1 libnuma-dev

# Configure 开启 cuda加速 动态链接 开启调试模式
./configure --enable-nonfree --enable-cuda-nvcc --enable-libnpp --extra-cflags=-I/usr/include --extra-ldflags=-L/usr/lib/x86_64-linux-gnu/ --disable-static --enable-shared --enable-debug=3 --disable-optimizations --disable-stripping
```

下图为支持 cuda 加速的 ffmpeg 帮助命令输出
![ffmpeg-out](ffmpeg-output.png)
