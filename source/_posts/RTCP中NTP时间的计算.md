---
title: RTCP中NTP时间的计算
tags:
  - rtcp
abbrlink: 89f0f1e3
date: 2023-07-12 17:19:28
---

# 如何计算ntp时间
NTP时间戳是用一个64bit无符号的定点数表示。它表示自从1900年1月1日00:00 到现在过了多少秒。
UTC表示自从1970年1月1日00:00 到现在过了多少秒。

NTP时间相对Unix时间有70年的偏移(包括17个闰年)
偏移值秒数: `(70 * 365 + 17) * 24 * 60 * 60 = 2208988800`
av_gettime获取unix时间的微妙值，加上NTP_OFFSET_US 就得到NTP时间的微妙值。
![NTP_OFFSET](1.png)
![ff_ntp_time](2.png)

# 如果将64位ntp拆分成两个32位
![4](4.png)