---
title: ffmpeg推送hls流
tags:
  - ffmpeg
abbrlink: ab268f66
date: 2023-06-01 13:12:58
---

HLS (HTTP Live Streaming)是Apple的动态码率自适应技术。

主要用于PC和Apple终端的音视频服务。

包括一个m3u(8)的索引文件，TS媒体分片文件和key加密串文件。

常用的流媒体协议主要有 HTTP 渐进下载和基于 RTSP/RTP 的实时流媒体协议，

这二种基本是完全不同的东西，目前比较方便又好用的是用 HTTP 渐进下载方法，

在这个中 apple 公司的 HTTP Live Streaming 是这方面的代表。它最初是苹果公司针对iPhone、iPod、iTouch和iPad等移动设备而开发的。现在见到在桌面也有很多应用了，HTML5 是直接支持这个。

下面介绍两条ffmpeg推送hls流的命令
1. vod模式

源文件包含一条音轨一条视轨

hls包含三个版本码流：

a、原始大小的视频，264编码，码率5M；音频双声道，aac编码，码率96k

b、720p的视频， 264编码，码率3M；音频双声道，aac编码，码率96k

c、360p的视频， 264编码，码率1M；音频双声道，aac编码，码率48k

```
ffmpeg -i .\in.mp4 \
-filter_complex "[0:v]split=3[v1][v2][v3];[v1]copy[v1out];[v2]scale=w=1280:h=720[v2out];[v3]scale=w=640:h=360[v3out]" \
-map [v1out] -c:v:0 libx264 -x264-params "nal-hrd=cbr:force-cfr=1" -b:v:0 5M -maxrate:v:0 5M -minrate:v:0 5M -bufsize:v:0 10M -preset slow -g 48 -sc_threshold 0 -keyint_min 48 \
-map [v2out] -c:v:1 libx264 -x264-params "nal-hrd=cbr:force-cfr=1" -b:v:1 3M -maxrate:v:1 3M -minrate:v:1 3M -bufsize:v:1 10M -preset slow -g 48 -sc_threshold 0 -keyint_min 48 \
-map [v3out] -c:v:2 libx264 -x264-params "nal-hrd=cbr:force-cfr=1" -b:v:2 1M -maxrate:v:2 1M -minrate:v:2 1M -bufsize:v:2 2M  -preset slow -g 48 -sc_threshold 0 -keyint_min 48\
-map a:0 -c:a:0 aac -b:a:0 96k -ac 2 \
-map a:0 -c:a:1 aac -b:a:1 96k -ac 2 \
-map a:0 -c:a:2 aac -b:a:2 48k -ac 2 \
-f hls \
-hls_time 2 \                                      #分片时长 单位秒
-hls_playlist_type vod \                           #playlist类型为vod
-hls_flags independent_segments \                  #分片以关键帧开始
-hls_segment_type mpegts \                         #分片用ts封装
-hls_segment_filename stream_%v/data%02d.ts \      #分片名称
-master_pl_name master.m3u8 \                      #需要master文件的可以指定此选项，文件包含子m3u8信息
-var_stream_map "v:0,a:0 v:1,a:1 v:2,a:2" \        #流映射，在某些情况下可以减少编码，复用已编码流
stream_%v.m3u8
```

2. live模式
   
将rtmp流转换成hls流，不转码
```
ffmpeg -i rtmp://10.132.95.12/live/ls_720p_H264_cs_720p \
-dn -c copy \
-f hls \
-hls_time 2 \                           #分片时长 单位秒
-hls_flags independent_segments \       #分片以关键帧开始
-hls_segment_type mpegts \              #分片用ts封装
-hls_list_size 3 \                      #playlist里包含ts的个数
-hls_segment_filename data%02d.ts \
-var_stream_map "v:0,a:0" \
stream.m3u8
```
