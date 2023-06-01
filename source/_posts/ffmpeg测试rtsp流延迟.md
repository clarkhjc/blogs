---
title: ffmpeg测试rtsp流延迟
date: 2023-06-01 13:09:05
tags:
- ffmpeg
---

RTSP是 TCP/IP 协议体系中的一个应用层协议，该协议定义了一对多应用程序如何有效地通过 IP 网络传送多媒体数据。RTSP在体系结构上位于RTP和RTCP之上，它使用TCP或UDP完成数据传输。HTTP与RTSP相比，HTTP传送HTML，而RTSP传送的是多媒体数据。

1. 安装rtspserver

>https://github.com/aler9/rtsp-simple-server/releases

下载对应系统的版本

在终端中执行
```
./rtsp-simple-server
```
![rtsp](rtsp.png)

2. 生成带时间戳的测试流

```
ffmpeg -y -re -f lavfi -i testsrc=size=1920x1080:rate=25 -vf "settb=AVTB,setpts='trunc(PTS/1K)*1K+st(1,trunc(RTCTIME/1K))-1K*trunc(ld(1)/1K)',drawtext=x=220:fontsize=120:text='%{localtime}.%{eif\:1M*t-1K*trunc(t*1K)\:d}'" -r:v 25 -f rtsp rtsp://localhost:8554/mystream
```
本地时间戳精确到毫秒

效果如下：
![s1](s1.png)

3. 用vlc播放rtsp流并和本地时间进行对比

![s2](s2.png)
延迟在1s左右。

