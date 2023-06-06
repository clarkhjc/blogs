---
title: ffmpeg libx264编码指南（补充）
tags:
  - ffmpeg-cmd
  - libx264
abbrlink: 40939fe6
date: 2023-06-01 12:06:49
---

- **CBR (Constant Bit Rate)**

没有真正的CBR模式，但是你可以通过调整ABR的参数来模拟固定码率编码。
```
ffmpeg -i input.mp4 -c:v libx264 -x264-params "nal-hrd=cbr" -b:v 1M -minrate 1M -maxrate 1M -bufsize 2M output.ts
```

-bufsize 是码率控制缓存，以上面命令行为例，编码器会在2M数据范围强制实现1M的平均码率。这里假设接收端/播放器会缓冲这么多数据，这意味着在这个范围内的波动是可以接受的。

- **Constrained encoding (VBV / maximum bit rate)**

如果您想限制最大码率，或者将流的码率限定在一定的范围内，请使用此模式。他对在线流媒体特别有用，因为客户端期望一个特定的平均码率。

在命令行中使用选项-maxrate 和 -bufsize，对于CRF编码和ABR编码都适用。
```
ffmpeg -i input -c:v libx264 -crf 23 -maxrate 1M -bufsize 2M output.mp4
```

编码器以crf 23为编码目标，如果输出码率大于1M，编码器会增大crf值防止码率飙高。但是，请注意libx264并没有严格控制最大码率，所以如果想获得完美的最大码率，使用two-pass编码：
```
ffmpeg -i input -c:v libx264 -b:v 1M -maxrate 1M -bufsize 2M -pass 1 -f mp4 /dev/null
ffmpeg -i input -c:v libx264 -b:v 1M -maxrate 1M -bufsize 2M -pass 2 output.mp4
```

- **Low Latency**

-tune zerolatency 选项可以实现低延迟编码

- **编码视频的一部分**

可以使用-ss和-t/-to选项

-ss: 开始偏移时间
-t: 时长
-to: 在指定位置停止输出
三个值的形式可以是秒或者HH:MM:SS格式。

- **对于在web端播放的视频启动faststart选项**

如果编码输出的是mp4文件，并且文件需要在浏览器中播放，那么可以在输出选项中增加-movflags +faststart，它会将一些信息移动到文件的开头，并允许视频在观看者完全下载之前开始播放。

- **OpenCL**

在命令行中添加`-x264-params opencl=true` 可以启动OpenCL lookahead，它将在不影响质量的情况下，使用GPU略微提高编码速度。
你会看到`Compiling OpenCL kernels...`或者`OpenCL acceleration enabled with NVIDIA ...`的打印，以及一个名为x264_lookahead.clbin文件，这个文件在编码结束后可以放心删除。

- **不同预设如何影响编码时间?**

这取决于源视频、目标码率和硬件配置。一般来说，码率越高，编码所需的时间就越多。

下面是一个例子，显示了1080p视频的two-pass编码时间：
![x264_preset](x264_preset.jpeg)

从medium到slow，时间增加了40%。
从medium到slower，时间增加100%。
与medium相比，veryslow需要280%的编码时间，但是相对于slower在质量方面只有很小的改进。
使用fast可以节省大约10%的编码时间，faster可以节省25%。ultrafast将节省55%，但是代价是质量要低得多。



