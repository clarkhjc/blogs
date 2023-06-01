---
title: ffmpeg命令行中stream specifier的用法
date: 2023-06-01 09:40:37
tags:
- ffmpeg
---

流说明符是一个字符串，通常附加在选项名称后面，并以冒号分隔。

流说明符的形式有：

- **stream_index**

通过索引匹配流，比如：

```
# -b:0 指定第一条流 -b:1 指定第二条流
./ffmpeg -i in.mp4 -b:0 2000k -b:1 256k -c:v libx264 -c:a aac out.mp4

# -c:a:0 aac 指定第一条音频流用aac编码 -c:a:1 ac3 指定第二条音频流用ac3编码
./ffmpeg -i in.mp4 -i in.mp3 -map 0 -map 1 -b:0 2000k -b:1 256k -b:2 384k -c:v libx264 -c:a:0 aac -c:a:1 ac3 out.mp4 -y
```

- **stream_type[:additional_stream_specifier]**

| 流类型 | stream_type |
|-------|-------------|
| video | v or V (v匹配所有视频流；V匹配没有附加图片、没有缩略图、没有封面的视频流) |
| audio | a |
| subtitle | s |
| data | d |
| attachments | t |

比如：

```
# -c:v 指定视频的编码器 -c:a 指定音频的编码器
./ffmpeg -i in.mp4 -b:0 2000k -b:1 256k -c:v libx264 -c:a aac out.mp4

# -c:a:0 aac 指定第一条音频流用aac编码 -c:a:1 ac3 指定第二条音频流用ac3编码
./ffmpeg -i in.mp4 -i in.mp3 -map 0 -map 1 -b:0 2000k -b:1 256k -b:2 384k -c:v libx264 -c:a:0 aac -c:a:1 ac3 out.mp4 -y
```

- **p:program_id[:additional_stream_specifier]**

匹配program id是program_id的流，比如：

```
# 0:p:keep_pid 保留program id为keep_pid的流，不匹配program id为first_remove_pid和second_remove_pid的流

./ffmpeg -i /dev/video0 -acodec copy -vcodec copy -map 0:p:keep_pid -map -0:p:first_remove_pid -map -0:p:second_remove_pid output.m2ts
```

- **#stream_id or i:stream_id：**

通过stream id匹配流（比如MPEG-TS中的PID）

```
# i:0x1102 指定stream id为0x1102的流

./ffmpeg -i sample.m2ts -map i:0x1102 -c copy out.ac3
```

- **m:key[:value]**

匹配有元数据标记key的流，如果指定key的值value，那么匹配key和value。

```
# 0:m:language:eng 选择语言是英文的音频流

ffmpeg -i INPUT -map 0:m:language:eng OUTPUT
```
