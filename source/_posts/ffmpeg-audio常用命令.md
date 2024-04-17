---
title: ffmpeg audio常用命令
tags:
  - ffmpeg
abbrlink: '47995412'
date: 2023-06-01 09:31:13
---

## 常用命令行参数：

-ar 设置采样率

-ac 设置声道数

-af 设置音频过滤器

-sample_fmt 设置采样格式，参见./ffmpeg -sample_fmts

-channel_layout 设置声道布局，参见./ffmpeg -layouts

## 常用命令行例子：
- pcm转wav
  
  `ffmpeg -f s16le -ar 44.1k -ac 2 -i file.pcm file.wav`

- 控制输出音频声道数、布局、采样率、采样格式

  ```
  ffmpeg -i 5.1.wav -ac 4 -ar 44100 -sample_fmt s16 out.wav

  // 查看信息如下
  // Stream #0:0: Audio: pcm_s16le ([1][0][0][0] / 0x0001), 44100 Hz, 4.0, s16, 2822 kb/s
  ```

  ```
  ffmpeg -i 5.1.wav -ac 4 -channel_layout quad -ar 44100 -sample_fmt s16 out.wav

  // 查看信息如下
  // Stream #0:0: Audio: pcm_s16le ([1][0][0][0] / 0x0001), 44100 Hz, quad, s16, 2822 kb/s
  ```

  ```
  ffmpeg -i 5.1.wav -ac 4 -channel_layout quad(side) -ar 44100 -sample_fmt s16 out.wav

  // 查看信息如下
  // Stream #0:0: Audio: pcm_s16le ([1][0][0][0] / 0x0001), 44100 Hz, quad(side), s16, 2822 kb/s
  ```
  
## pan过滤器

  语法 pan=输出声道布局｜某一个输出声道计算表达式｜某一个输出声道计算表达式。。。

  输出声道布局可以设置为mono、stereo、5.1等，具体参见./ffmpeg -layouts

  输出声道计算表达式：

  `"out_name=[gain*]in_name[(+-)[gain*]in_name...]"`

  out_name和in_name可以是声道名称，比如FL、FR等，或者为声道索引比如c0、c1等。

  1. stereo下采样到mono，左声道的权重更大
  
  `-af "pan=1c|c0=0.9*c0+0.1*c1"`

  2. 源是5.1声道，通过丢弃其他声道的方式获得stereo
  
  `-af "pan=stereo|c0=FL|c1=FR"`

  3. 源是5.1声道，交换FL和FR，其他声道不变

  `-af "pan=5.1|c0=c1|c1=c0|c2=c2|c3=c3|c4=c4|c5=c5"`

  4. 源是stereo，设置FL静音

  `-af "pan=stereo|c1=c1"`

  5. 源是stereo，输出左右声道都为源的右声道

  `-af "pan=stereo|c0=FR|c1=FR"`

更详细命令行见
>https://trac.ffmpeg.org/wiki/AudioChannelManipulation

