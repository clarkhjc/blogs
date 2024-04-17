---
title: ffmpeg libx264编码指南
tags:
  - ffmpeg
  - libx264
abbrlink: dd9a4102
date: 2023-06-01 12:24:34
---

首先在系统上安装x264编码器，配置ffmpeg时打开--enable-libx264选项，就可以在ffmpeg中使用libx264了。
通常推荐使用的码率控制模式有两种，Constant Rate Factor (CRF) 和 Two-Pass ABR。
码率控制决定了每一帧可以使用多少比特，这将决定编码后文件大小以及文件质量。

# Constant Rate Factor (CRF)

如果你想保证最好的质量，并且不太关心文件大小，那么你可以使用这个模式。这也是大部分情况下推荐的模式。

这种方法允许编码器在输出文件大小不太重要的情况下，尝试为整个文件实现固定输出质量。

通过为每一帧调整量化参数，它得到所需的比特率，以保证所要求的编码质量。

缺点是你不能获取特定的文件大小或控制码率，这意味着这种模式不能用于流媒体视频编码。

## 选择CRF值

CRF范围是0-51，值越小质量越好。

| CRF值 | 说明 | 
| ----  | ----|
|0 | 无损编码 |
|17~28 | 常用值范围 17、18：可以保证在视觉上无损(不是技术上无损) 23：默认值 |
|51 | 产生最差的质量|

CRF值对质量的影响是指数级的，CRF每增加6，码率大概下降一半，CRF每减小6，码率大概增加一倍。

选择CRF值的原则是：选择一个相对较大的CRF值，如果输出质量很好，可以试着增大CRF值，如果质量糟糕，试着减小CRF值。

### 选择preset和tune

#### preset

预设是提供特定编码速度与压缩比的选项的集合。速度越慢的预设提供越好的压缩，这意味着在相同码率情况下，越慢的预设提供越好的质量；在固定质量的情况下，越慢的预设提供越小的码率。

可用的预设值按速度降序排列为：

  - ultrafast
  - superfast
  - veryfast
  - faster
  - fast
  - medium – 默认预设
  - slow
  - slower
  - veryslow
  - placebo


#### tune
  
您可以选择使用-tune来根据您的输入细节更改设置，当前tune包括:

  - film – 用于高质量的电影内容
  - animation – 适合动画片
  - grain – 保留纹理结构，适合老旧电影
  - stillimage – 适合制作类似幻灯片的内容
  - fastdecode – 通过禁用某些过滤器允许更快的解码
  - zerolatency – 适合快速编码和低延迟流
  
如果您不确定使用什么，或者您的输入不匹配任何tune，则省略-tune选项。

#### profile

选项-profile:v将输出限制为特定的H.264档次，通常不需要使用这个选项，建议也是不要设置profile，这将允许x264自动选择适当的profile。

一些设备(大多非常旧或过时)只支持Baseline或Main profile。您可以使用-profile:v baseline或-profile:v main来设置这些profile。大多数现代设备支持更先进的high profile。

x264支持的profile包括：

- baseline
- main
- high
- high10 (第一个兼容10bit的profile)
- high422 (支持 yuv420p, yuv422p, yuv420p10le 和 yuv422p10le)
- high444 (支持上述以及 yuv444p 和 yuv444p10le)

#### 列举presets和tunes

列举所有可能的内部presets和tunes
```
ffmpeg -hide_banner -f lavfi -i nullsrc -c:v libx264 -preset help -f mp4 -
```
注意：Windows用户需要使用NUL而不是-作为输出

## CRF 举例

```
ffmpeg -i input -c:v libx264 -preset slow -crf 22 -c:a copy output.mkv
```
该命令编码的视频质量较好，使用较慢的预设，以达到更好的压缩。请注意，在这个例子中，输入文件的音频流只是简单地复制到输出，而不是重新编码。如果你正在编码一组相似的视频，对所有视频应用相同的设置:这将确保它们都具有相似的质量。

# Two-Pass

如果您的目标是特定的输出文件大小，并且帧与帧之间的输出质量不那么重要，请使用此码率控制模式。

## Two-Pass 举例

对于two-pass，你需要运行ffmpeg两次，使用几乎相同的设置，除了:

- 第一次使用-pass 1、第二次使用-pass 2。
- 第一次输出是一个空的文件描述符，不是一个实际的文件（会产生一个日志文件，ffmpeg在第二次编码时会使用）。
- 在第一次编码时，可以通过指定-an选项忽略音频处理。

```
ffmpeg -y -i input -c:v libx264 -b:v 2600k -pass 1 -an -f null /dev/null
ffmpeg -i input -c:v libx264 -b:v 2600k -pass 2 -c:a aac -b:a 128k output.mp4
```
注意：Windows用户应该使用NUL而不是/dev/null

# 无损 H.264

使用-qp 0选项获得无损视频，对于profile是High 4:4:4 Predictive的编码使用-crf 0获得无损视频。

快速编码例子：
```
ffmpeg -i input -c:v libx264 -preset ultrafast -qp 0 output.mkv
```

更好压缩的例子：
```
ffmpeg -i input -c:v libx264 -preset veryslow -qp 0 output.mkv
```

# 覆盖默认预设设置

你可以用x264-params选项覆盖preset的默认设置值。一般不建议这样做，因为预设值是由x264开发人员创建的，调整值以获得更好的输出通常是浪费时间。

例子：
```
ffmpeg -i input -c:v libx264 -preset slow -crf 22 -x264-params keyint=123:min-keyint=20 -c:a copy output.mkv
```
警告:不要使用x264opts选项，因为它最终会被删除。请改用x264-params。


