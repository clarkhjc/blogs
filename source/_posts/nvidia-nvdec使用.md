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