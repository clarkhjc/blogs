---
title: ffmpeg命令行耗时输出(benchmark)
date: 2023-06-01 12:45:02
tags:
- ffmpeg
---

ffmpeg命令行支持输出程序耗时统计，默认是不打印的，需要通过参数控制

`-benchmark (global)`

在转码结束后显示统计信息。展示的信息包括实际耗时、系统耗时、用户耗时以及最大内存使用。我们一般关心的是实际耗时。

`-benchmark_all (global)`

在编码过程中展示统计信息。音频、视频的每一帧解码和编码都会打印，输出信息非常多。

比如命令行
```
./ffmpeg -i in.MXF -benckmark out.mp4 -y
```
![benchmark-1](benchmark-1.png)
整个转码耗时25.803s，最大内存使用217372kB。

接下来查看一下源码：
![benchmark-2](benchmark-2.png)
截图右边是统计和打印的地方，在transcode()函数前后进行耗时统计并打印。

BenchmarkTimeStamps结构体包含三个字段:
real_usec：实际耗时，精确到微妙。通过获取系统当前时间（比如gettimeofday）得到。
user_usec：用户耗时，精确到微妙。通过getrusage(linux)或者GetProcessTimes(win)得到。
sys_usec ：系统耗时，精确到微妙。通过getrusage(linux)或者GetProcessTimes(win)得到。