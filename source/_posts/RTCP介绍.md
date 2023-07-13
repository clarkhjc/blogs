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
- 前两个功能要求所有参与者都发送RTCP数据包，因此为了使RTP扩展到大量参与者，必须控制速率。通过让每个参与者将其控制数据包发送给所有其他参与者，每个参与者都可以独立地观察参与者的数量。
- 第四个可选功能是传递最少的会话控制信息，例如要在用户界面中显示的参与者标识

# RTCP包格式
## RTCP包类型
- SR：发送方报告
- RR：接收方报告
- SDES：源描述项，包括CNAME等项
- BYE：表示参与结束
- APP：特定于应用程序的功能

### SR包格式
![sr](sr.png)

- 版本（V）：2比特，RTCP版本。
- 填充（P）：1比特，如果该位置为1，则该RTCP包的尾部就包含附加的填充字节。
- 接收报告计数器（RC）：5比特，该SR包中的接收报告块的数目，可以为零。
- 包类型（PT）：8比特，SR包是200。
- 长度域（Length）：16比特，RTCP包的长度，包括头和填充的内容。
- 同步源（SSRC of sender）：32比特，SR包发送者的同步源标识符。与对应RTP包中的SSRC一样。
- NTP timestamp（MSW+LWS）：64比特， 表示发送此报告时以挂钟时间测量的时间点。 结合来自各个接收器的接收报告中返回的时间戳，它可用于估计往返于接收器的往返传播时间。
- RTP timestamp：32比特，与NTP时间戳对应，与RTP数据包中的RTP时间戳具有相同的单位和随机初始值。
- Sender’s packet count：32比特，从开始发送包到产生这个SR包这段时间里，发送者发送的RTP数据包的总数. SSRC改变时，这个域清零。
- Sender`s octet count：32比特，从开始发送包到产生这个SR包这段时间里，发送者发送的净荷数据的总字节数（不包括头部和填充）。发送者改变其SSRC时，这个域要清零。
- SSRC_n ：32比特，此接收报告块中信息所属源的SSRC标识符
- 丢失率(Fraction Lost):8比特，表明从上一个SR或RR包发出以来从同步源n(SSRC_n)来的RTP数据包的丢失率
- 累计的包丢失数目（cumulative number of packets lost C ）：24比特，从开始接收到SSRC_n的包到发送SR,从SSRC_n传过来的RTP数据包的丢失总数。
- 收到的扩展最大序列号(extended highest sequence number received  EHSN )：从SSRC_n收到的RTP数据包中最大的序列号
- 接收抖动（Interarrival jitter）：32比特，RTP数据包接受时间的统计方差估计
- 上次SR时间戳（Last SR,LSR）：32比特，取最近从SSRC_n收到的SR包中的NTP时间戳的中间32比特。如果目前还没收到SR包，则该域清零
- 上次SR以来的延时（Delay since last SR,DLSR）：32比特，上次从SSRC_n收到SR包到发送本报告的延时
- 扩展字段 profile-specific extensions

### SDES包格式
![sdes](sdes.png)
- SDES包是一个三层结构，由一个报头和零个或多个chunk组成。
- 一个chunk由一个SSRC标识符和0个或多个item组成。每个chunk从32位边界开始。
- 每个item由一个字节的类型字段、一个字节的长度字段和item内容组成。长度字段的值不包括类型和长度字段两个字节。
- item内容长度不超过255字节。
- 多个item是连续的，即，单个item不会填充到32位边界。
- 一个chunk中的items以一个或者多个0字节结尾。
- item有8种，只有cname是必须要的，其他可选。

#### cname语法
![cname](cname.png)

### BYE包格式
![bye](bye.png)



