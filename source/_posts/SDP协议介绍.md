---
title: SDP协议介绍
tags:
  - sdp
  - rtsp
  - ffmpeg
abbrlink: a357fdc5
date: 2023-07-07 11:55:15
---
协议

> RFC2327

# SDP协议作用

- SDP用于描述多媒体会话，用于会话通知、会话邀请和其他形式的多媒体会话发起。
- SDP纯粹是一种用于会话描述的格式，它不包含传输协议，它可以和不同的传输协议配合使用，
  包括会话声明协议(Session Announcement Protocol)、会话发起协议(Session Initiation Protocol)、
  实时流协议(Real-Time Streaming Protocol)、使用MIME扩展的电子邮件和HTTP协议。

# SDP结构

一个SDP会话描述由多个文本行组成，每一行的形式是`<type>`=`<value>`。

- `<type>`由一个字符构成
- `<value>`的格式取决于`<type>`
- =号两边不允许有空格
- `<value>`中多个字段通过单个空格分隔。

一个会话描述由一个会话层描述和可选的多个媒体层描述构成。

- 会话层以 `v=`行开头，持续到第一个媒体层部分。
- 媒体层以 `m=`行开头，持续到下一个媒体层描述或者到整个描述的结尾。

# SDP的type值

## 会话描述

    v=  (protocol version)
    o=  (owner/creator and session identifier).
    s=  (session name)
    i=* (session information)
    u=* (URI of description)
    e=* (email address)
    p=* (phone number)
    c=* (connection information - not required if included in all media)
    b=* (bandwidth information)
    One or more time descriptions (see below)
    z=* (time zone adjustments)
    k=* (encryption key)
    a=* (zero or more session attribute lines)
    Zero or more media descriptions (see below)

## 时间描述

    t=  (time the session is active)
    r=* (zero or more repeat times)

## 媒体描述

    m=  (media name and transport address)
    i=* (media title)
    c=* (connection information - optional if included at session-level)
    b=* (bandwidth information)
    k=* (encryption key)
    a=* (zero or more media attribute lines)

# 一个SDP例子

```
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

![sdp-sample](sdp-sample.png)

# type和value介绍

1. v: Protocol Version
   `v=0`
   指定会话描述协议的版本号，一般为0
2. o: Origin
   `o=<username> <session id> <version> <network type> <address type> <address>`
   指定会话的发起人，包括他们的用户名、主机地址、会话id、会话版本号。

- username：用户登陆名称，如果源主机不支持用户id的概念，则为-。
- session id：数字字符串。
- version：会话版本号。
- network type：网络类型，比如IN，表示因特网。
- address type：地址类型，比如IP4、IP6。
- address：机器的IP地址。

3. s: Session Name
   `s=<session name>`
   会话名称，一个会话描述有且只有一个此字段。
4. c: Connection Data
   `c=<network type> <address type> <connection address>`
   包含连接数据。

- network type：网络类型，比如IN，表示因特网。
- address type：地址类型，比如IP4、IP6。
- connection address：通信的地址。

5. t: Times
   `t=<start time>  <stop time>`
   指定会议会话的开始和结束时间。值为NTP时间值的十进制形式，以秒为单位。要将这些值转换为UNIX时间，减去十进制2208988800。
   如果stop time设置为零，则会话不受限制，尽管它在start time之后才激活。如果start-time也为0，则认为该会话是永久的。
6. a: Attributes

```
a=<attribute>
a=<attribute>:<value>
```

属性是扩展SDP功能的主要手段。属性分为会话层属性、媒体层属性。
属性字段有两种形式，
第一种为 `a=<attribute>`, 比如 `a=recvonly`
第二种为 `a=<attribute>:<value>`, 比如 `a=orient:landscape`

7. b: Bandwidth
   `b=<modifier>:<bandwidth-value>`
   指定会话或媒体要使用的建议带宽，并且是可选的.
   `<bandwidth-value>`单位是kbps
   `<modifier>`有两个预定义的值

- CT: 给出所有站点上所有媒体的总带宽数字.
- AS: 给出单个站点上单个媒体的带宽数字.

8. m: Media Announcements
`m=<media> <port> <transport> <fmt list>`
- media: 指定媒体类型，值有"audio", "video", "application", "data" 和 "control"
- port: 传输端口号
- transport: 传输协议，值有RTP/AVP、udp
- fmt list: 指定媒体格式，取值可以参看ffmpeg中`ff_rtp_get_payload_type`函数。

# 实现参考
ffmpeg源码libavformat/sdp.c文件的av_sdp_create函数。
