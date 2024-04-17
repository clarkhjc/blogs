---
title: nvidia nvdec使用
abbrlink: 4eb6e8e
date: 2024-04-15 16:11:04
tags:
    - nvidia
---

# cuda和显卡驱动安装
参考
>https://clarkhjc.gitee.io/clarkhjc/2024/04/17/ffmpeg%E7%BC%96%E8%AF%91%E6%94%AF%E6%8C%81Nvidia%E5%8A%A0%E9%80%9F/

# nvdec头文件和库文件

## 头文件
- **sdk中的头文件**
nvidia decoder api包含在cuviddec.h和nvcuvid.h两个头文件中，文件位于sdk的`Interface`目录中。
<br>
- **ffmpeg中的头文件**
ffmpeg单独维护了一个仓库[nv-codec-headers](https://git.videolan.org/git/ffmpeg/nv-codec-headers.git)，用于存放nvidia解码和编码所需的头文件，解码用到`dynlink_cuviddec.h`和`dynlink_nvcuvid.h`，这里的头文件采用的是动态加载库方式。

## 库文件
- 显卡驱动安装后，会在系统目录安装`libnvcuvid.so`库，在ubuntu系统中的路径为`/usr/lib/x86_64-linux-gnu/libnvcuvid.so`，这是解码需要依赖的库。

# VIDEO_CODEC_SDK使用
- sdk下载地址
>https://developer.nvidia.com/nvidia-video-codec-sdk/download

- 安装sdk依赖项
```shell
# ffmpeg
自行编译安装
# vulkan
sudo apt-get install libvulkan-dev
# EGL
sudo apt-get install libegl1
# FREEGLUT
sudo apt-get install freeglut3-dev
# GLEW
sudo apt-get install libglew-dev

# GCC-10
sudo apt-get install gcc-10 g++-10
```
- 下载解压之后，进入`Video_Codec_SDK_12.2.72/Samples`目录，执行
```shell
# 首先设置gcc使用gcc10
export CC=/usr/bin/gcc-10
export CXX=/usr/bin/g++-10

mkdir build
cd build
cmake ..
make
```
