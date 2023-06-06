---
title: ffmpeg命令行参数解析-split_commandline函数
tags:
  - ffmpeg-src
abbrlink: 6d81cb11
date: 2023-06-01 12:39:36
---

# 选项解析结构体介绍
选项解析涉及几个结构体：OptionParseContext、OptionGroupList、OptionGroupDef、OptionGroup、Option、OptionDef。
结构体包含关系如下：
![选项解析涉及的结构体](选项解析涉及的结构体.png)

OptionParseContext包含三部分：
- global_opts（类型OptionGroup）存放所有全局选项，比如-y
- groups[0]（类型OptionGroupList）包含所有输出选项
- groups[1]（类型OptionGroupList）包含所有输入选项

OptionGroupList：
- groups[0]（类型OptionGroup）第一个文件的选项组
- groups[1]（类型OptionGroup）第二个文件的选项组
- ......

OptionGroup选项组包含一个文件的所有选项：
- opts[0]（类型Option）第一个选项
- opts[1]（类型Option）第二个选项
- ......

ffmpeg支持输入多个文件输出多个文件，比如输入in1.mp4、in2.mp4；输出out1.mp4、out2.mp4
```shell
./ffmpeg -ss 00:00:10 -i in1.mp4 -ss 00:00:20 -i in2.mp4 -map 0:v -map 1:a -c:v libx264 -c:a aac out1.mp4 -map 1:v -map 0:a -c:v mpeg4 -c:a ac3 out2.mp4 -y
```
![上图命令行拆分后存放结构](multi-in_multi-out.png)

# 选项拆分函数

**函数原型**
```c
int split_commandline(OptionParseContext *octx, int argc, char *argv[],
                      const OptionDef *options,
                      const OptionGroupDef *groups, int nb_groups);
```
将命令行拆分后存放在OptionParseContext结构体中，方便后续处理。

**调用**

![ffmpeg_parser_options](ffmpeg_parser_options.png)
第四个参数传递的是options，这是一个预定义的OptionDef选项数组。

第五个参数groups定义如下
```c
static const OptionGroupDef groups[] = {
    [GROUP_OUTFILE] = { "output url",  NULL, OPT_OUTPUT },
    [GROUP_INFILE]  = { "input url",   "i",  OPT_INPUT },
};
```
说明有两个选项组，一个包含输出参数，一个包含输入参数。


下面对函数源码进行简单的注释
![split_commandline-1](split_commandline-1.png)
![split_commandline-2](split_commandline-2.png)
![split_commandline-3](split_commandline-3.png)

