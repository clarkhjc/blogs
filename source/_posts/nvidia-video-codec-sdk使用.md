---
title: nvidia video_codec_sdk使用
tags:
  - nvidia
abbrlink: e151a831
date: 2024-04-17 16:44:26
---

# cuda和显卡驱动安装
参考
>https://clarkhjc.gitee.io/clarkhjc/2024/04/17/ffmpeg%E7%BC%96%E8%AF%91%E6%94%AF%E6%8C%81Nvidia%E5%8A%A0%E9%80%9F/

# sdk头文件和库文件

## 头文件
- **sdk中的头文件**
nvidia decoder api包含在`cuviddec.h`和`nvcuvid.h`两个头文件中，文件位于sdk的`Interface`目录中。
nvidia encoder api包含在`nvEncodeAPI.h`文件中，文件位于sdk的`Interface`目录中。

## 库文件
- 显卡驱动安装后，会在系统目录安装解码和编码的库。
解码库为`libnvcuvid.so`库，在ubuntu系统中的路径为`/usr/lib/x86_64-linux-gnu/libnvcuvid.so`。
编码库为`libnvidia-encode.so`, 在ubuntu系统中的路径为`/usr/lib/x86_64-linux-gnu/libnvidia-encode.so`。

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
