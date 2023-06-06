---
title: ffmpeg中数字选项的单位介绍
tags:
  - ffmpeg-cmd
abbrlink: 7f420acd
date: 2023-06-03 08:45:46
---

B: 8 bits
K: 1000 bits
M: 1000 * 1000 bits
G: 1000 * 1000 * 1000 bits

KB: 1000 * 8 bits
MB: 1000 * 1000 * 8 bits
GB: 1000 * 1000 * 1000 bits * 8 bits

KiB: 1024 * 8bits
MiB: 1024 * 1024 * 8bits
GiB: 1024 * 1024 * 1024 * 8bits

比如设置264编码码率
```shell
# 2000 kb/s
./ffmpeg -i in.mp4 -an -c:v libx264 -b:v 2000K -s 1280x720 out.mp4 -y

# 16000 kb/s
./ffmpeg -i in.mp4 -an -c:v libx264 -b:v 2000KB -s 1280x720 out.mp4 -y

# 16384 kb/s
./ffmpeg -i in.mp4 -an -c:v libx264 -b:v 2000KiB -s 1280x720 out.mp4 -y
```
