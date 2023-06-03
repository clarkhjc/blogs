---
title: ffmpeg-filter命令行介绍
date: 2023-06-03 08:28:52
tags:
- ffmpeg-cmd
---

一个filter可以有多个输入和多个输出，我们用下面的filtergraph来演示filter的用法：
![split](split.png)

split将输入分成两条流，分别用标签main和tmp表示；tmp经过crop裁剪和vflip垂直翻转；最终main和处理过的tmp进行叠加。

```shell
ffmpeg -i INPUT -vf "split [main][tmp]; [tmp] crop=iw:ih/2:0:0, vflip [flip]; [main][flip] overlay=0:H/2" OUTPUT
```
上面的命令实现视频的上半部分和下半部分镜像对称。
![split_show](split_show.png)

filter可以组成filter chain，中间用***逗号***隔开，比如`[tmp] crop=iw:ih/2:0:0,vflip [flip]`
filter chain之间用***分号***隔开，比如`split [main][tmp]; [tmp] crop=iw:ih/2:0:0, vflip [flip]; [main][flip] overlay=0:H/2`
filter名称后面可以加一个***等号***，等号后面是参数，参数之间用***冒号***分隔。
![filtergraph](filtergraph.png)

利用graph2dot将filtergraph转换成点语言的文本表示
1. 安装graphviz，会用到dot命令
2. 在命令行的filtergraph前后加上nullsrc和nullsink，用逗号分隔
3. echo graph_description | graph2dot -o graph.tmp
4. dot -Tpng graph.tmp -o graph.png

```shell
echo "nullsrc=s=1920x1080:r=25:duration=10,split[main][tmp];[tmp]crop=iw:ih/2:0:0,vflip[flip];[main][flip]overlay=0:H/2,nullsink" | graph2dot -o graph.tmp && dot -Tpng graph.tmp -o graph.png
```

生成png如下：
![graph](graph.png)

- 视频加水印

```shell
ffmpeg -i input -i logo -filter_complex 'overlay=x=0:y=0' output
```

- 视频增加帧号

```shell
ffmpeg -re -i 1.mp4 -vf "drawtext=fontfile=/Windows/Fonts/arial.ttf:fontsize=160:text='%{frame_num}'" -c:v libx264 -c:a aac -f flv rtmp://localhost:1935/live/room
```

- negate

实现视频反转，比如黑色区域变成白色，白色区域变成黑色
```shell
ffmpeg -i in.mkv -vf negate tmp.mp4
```
原图：
![negate_1](negate_1.png)
反转后的图：
![negate_2](negate_2.png)

- hflip

实现视频水平翻转
```shell
ffmpeg -i in.mkv -vf negate tmp.mp4
```
![hflip](hflip.png)

- edgedetect

边缘检测
```shell
ffmpeg -i in.mkv -vf edgedetect tmp.mp4
```
原图:
![edgedetect_1](edgedetect_1.png)
检测之后的图:
![edgedetect_2](edgedetect_2.png)


