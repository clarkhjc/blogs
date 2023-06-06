---
title: ffmpeg ts文件截图分析
tags:
  - ffmpeg-src
abbrlink: 7571f9ae
date: 2023-06-01 13:05:40
---

调用ffmpeg api截图ts文件时遇到一个问题，seek不到关键帧。
调用av_seek_frame函数，flags传递的是AVSEEK_FLAG_BACKWARD，
希望可以定位到距离seek点最近的关键帧，但是接着通过av_read_frame函数
读取数据时，发现读到的是seek位置的帧（非关键帧）。

但是用ffmpeg命令行截图却可以截到指定位置的图片。
（ts文件是h264编码，gop是75，帧率25，我指定截取第4s位置的帧，也就是第100帧。）
调试了一下ffmpeg代码，在open_input_file函数中有一段代码，
![snapshot-1](snapshot-1.png)

这段代码是进行文件seek用的，调用的是avformat_seek_file函数。从ic->iformat->flags不包含AVFMT_SEEK_TO_PTS标志可以看出，ts不支持基于pts的seek。

接着在decode_video函数中解码视频帧的地方进行调试
![snapshot-2](snapshot-2.png)
第一次解码帧的时间戳是ts文件的首帧时间戳，所以可以看出ffmpeg命令行进行ts截图是从ts文件的首帧开始解码，一直解码到需要截图的帧。
