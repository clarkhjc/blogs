---
title: RTCP介绍
tags:
  - rtcp
  - rtsp
abbrlink: 1d1d1da0
date: 2023-07-10 15:42:25
---

参考
>https://blog.csdn.net/Chiang2018/article/details/122743685

协议
>RFC3550

# RTCP介绍
RTCP是Real-Time Transport Control Protocol的缩写，是基于周期性地向会话的所有参与者发送控制报文的协议，采用与数据包相同的分发机制。底层协议必须提供数据和控制数据包的多路复用，例如使用分开的UDP端口号。RTCP有四个功能：
- 提供分发数据质量的反馈
- RTCP为RTP源携带一个持久的传输级标识符，称为规范名称或CNAME。如果发现冲突或程序重新启动，SSRC标识符可能会更改，因此接收方需要CNAME来跟踪每个参与者
- 前两个功能要求所有参与者都发送RTCP数据包，因此为了使RTP扩展到大量参与者，必须控制速率。通过让每个参与者将其控制数据包发送给所有其他参与者，每个参与者都可以独立地观察参与者的数量。这个数字用于计算数据包发送的速率
- 第四个可选功能是传递最少的会话控制信息，例如要在用户界面中显示的参与者标识

# RTCP包格式
## RTCP包类型
- SR：发送方报告，用于从活动发送方参与者处发送和接收的统计信息
- RR：接收方报告，用于来自非活动发送方的参与者的接收统计数据，并与SR结合使用，用于报告超过31个来源的活动发送方
- SDES：源描述项，包括CNAME
- BYE：表示参与结束
- APP：特定于应用程序的功能

### SR包格式
![sr](sr.png)

- 版本（V）：2比特，RTCP版本。
- 填充（P）：1比特，如果该位置为1，则该RTCP包的尾部就包含附加的填充字节。
- 接收报告计数器（RC）：5比特，该SR包中的接收报告块的数目，可以为零。
- 包类型（PT）：8比特，SR包是200。
- 长度域（Length）：16比特，RTCP包的长度，包括填充的内容。
- 同步源（SSRC of sender）：32比特，SR包发送者的同步源标识符。与对应RTP包中的SSRC一样。
- NTP timestamp（MSW+LWS）：64比特， 表示发送此报告时以挂钟时间测量的时间点。 结合来自各个接收器的接收报告中返回的时间戳，它可用于估计往返于接收器的往返传播时间。
- RTP timestamp：32比特，与NTP时间戳对应，与RTP数据包中的RTP时间戳具有相同的单位和随机初始值。
- Sender’s packet count：32比特，从开始发送包到产生这个SR包这段时间里，发送者发送的RTP数据包的总数. SSRC改变时，这个域清零。
- Sender`s octet count：32比特，从开始发送包到产生这个SR包这段时间里，发送者发送的净荷数据的总字节数（不包括头部和填充）。发送者改变其SSRC时，这个域要清零。
- SSRC_n ：32比特，在此块中报告其接收的发送者的 SSRC 标识符
- 丢失率(Fraction Lost):8比特，表明从上一个SR或RR包发出以来从同步源n(SSRC_n)来的RTP数据包的丢失率
- 累计的包丢失数目（cumulative number of packets lost C ）：24比特，从开始接收到SSRC_n的包到发送SR,从SSRC_n传过来的RTP数据包的丢失总数。
- 收到的扩展最大序列号(extended highest sequence number received  EHSN )：从SSRC_n收到的RTP数据包中最大的序列号
- 接收抖动（Interarrival jitter）：32比特，RTP数据包接受时间的统计方差估计
- 上次SR时间戳（Last SR,LSR）：32比特，取最近从SSRC_n收到的SR包中的NTP时间戳的中间32比特。如果目前还没收到SR包，则该域清零
- 上次SR以来的延时（Delay since last SR,DLSR）：32比特，上次从SSRC_n收到SR包到发送本报告的延时
- 扩展字段 profile-specific extensions

