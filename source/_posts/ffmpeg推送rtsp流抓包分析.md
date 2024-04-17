---
title: ffmpeg推送rtsp流抓包分析
tags:
  - ffmpeg
abbrlink: 69c2bf6a
date: 2023-07-04 13:21:01
---

相关协议:
>rtsp: RFC 2326
>sdp: RFC 2327

推流命令:
```shell
ffmpeg -stream_loop 1 -re -i input.mp4 -c copy -f rtsp -rtsp_transport tcp rtsp://127.0.0.1:8554/rtsp_test
```

# OPTIONS方法
查询服务器支持哪些method

## 请求报文

```
OPTIONS rtsp://127.0.0.1:8554/rtsp_test RTSP/1.0
CSeq: 1
User-Agent: Lavf60.9.100
```

## 响应报文

```
RTSP/1.0 200 OK
CSeq: 1
Public: DESCRIBE, ANNOUNCE, SETUP, PLAY, RECORD, PAUSE, GET_PARAMETER, TEARDOWN
Server: gortsplib
```
ffmpeg实现代码：
file: rtsp.c
```c
int ff_rtsp_connect(AVFormatContext *s) {
    ...
    ff_rtsp_send_cmd(s, "OPTIONS", rt->control_uri, cmd, reply, NULL);
    ...
}
```

# ANNOUNCE方法
从客户端发往服务器时，传递媒体对象的描述信息，比如下面请求报文的Content部分包含音视频的SDP信息；
从服务器发往客户端时，ANNOUNCE实时更新会话描述信息。

## 请求报文

```
ANNOUNCE rtsp://127.0.0.1:8554/rtsp_test RTSP/1.0
Content-Type: application/sdp
CSeq: 2
User-Agent: Lavf60.9.100
Content-Length: 543

v=0
o=- 0 0 IN IP4 127.0.0.1
s=Which video network streaming protocol should I use?
c=IN IP4 127.0.0.1
t=0 0
a=tool:libavformat 60.9.100
m=video 0 RTP/AVP 96
b=AS:909
a=rtpmap:96 H264/90000
a=fmtp:96 packetization-mode=1; sprop-parameter-sets=Z01AH+ygKALdgIgAAAMACAAAAwGQeMGMsA==,aO+8gA==; profile-level-id=4D401F
a=control:streamid=0
m=audio 0 RTP/AVP 97
b=AS:69
a=rtpmap:97 MPEG4-GENERIC/48000/1
a=fmtp:97 profile-level-id=1;mode=AAC-hbr;sizelength=13;indexlength=3;indexdeltalength=3; config=118856E500
a=control:streamid=1
```

## 响应报文

```
RTSP/1.0 200 OK
CSeq: 2
Server: gortsplib
```

ffmpeg实现代码:
file: rtspenc.c
```c
int ff_rtsp_setup_output_streams(AVFormatContext *s, const char *addr) {
    ...
    ff_rtsp_send_cmd_with_content(s, "ANNOUNCE", rt->control_uri,
                                  "Content-Type: application/sdp\r\n",
                                  reply, NULL, sdp, strlen(sdp));
    ...
}
```

# SETUP方法
指定发送媒体数据的传输机制，比如采用TCP还是UDP，端口是多少等。

## 视频流请求报文
```
SETUP rtsp://127.0.0.1:8554/rtsp_test/streamid=0 RTSP/1.0
Transport: RTP/AVP/TCP;unicast;interleaved=0-1;mode=record
CSeq: 3
User-Agent: Lavf60.9.100
```

## 视频流响应报文

```
RTSP/1.0 200 OK
CSeq: 3
Server: gortsplib
Session: d805ea37-b41f-4f24-9e77-9cd0970929c7
Transport: RTP/AVP/TCP;unicast;interleaved=0-1
```

## 音频流请求报文
```
SETUP rtsp://127.0.0.1:8554/rtsp_test/streamid=1 RTSP/1.0
Transport: RTP/AVP/TCP;unicast;interleaved=2-3;mode=record
CSeq: 4
User-Agent: Lavf60.9.100
Session: d805ea37-b41f-4f24-9e77-9cd0970929c7
```

## 音频流响应报文

```
RTSP/1.0 200 OK
CSeq: 4
Server: gortsplib
Session: d805ea37-b41f-4f24-9e77-9cd0970929c7
Transport: RTP/AVP/TCP;unicast;interleaved=2-3
```

ffmpeg实现代码:
file: rtsp.c
```c
int ff_rtsp_make_setup_request(AVFormatContext *s, const char *host, int port,
                              int lower_transport, const char *real_challenge) {
    ...
    ff_rtsp_send_cmd(s, "SETUP", rtsp_st->control_url, cmd, reply, NULL);
    ...
}
```

# RECORD方法
告知服务器准备录制一段媒体数据。

## 请求报文
```
RECORD rtsp://127.0.0.1:8554/rtsp_test RTSP/1.0
Range: npt=0.000-
CSeq: 5
User-Agent: Lavf60.9.100
Session: d805ea37-b41f-4f24-9e77-9cd0970929c7
```

## 响应报文
```
RTSP/1.0 200 OK
CSeq: 5
Server: gortsplib
Session: d805ea37-b41f-4f24-9e77-9cd0970929c7
```

ffmpeg代码实现:
file: rtspenc.c
```c
static int rtsp_write_record(AVFormatContext *s)
{
    ...
    ff_rtsp_send_cmd(s, "RECORD", rt->control_uri, cmd, reply, NULL);
    ...
}
```