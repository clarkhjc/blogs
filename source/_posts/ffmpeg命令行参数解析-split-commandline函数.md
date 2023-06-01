---
title: ffmpeg命令行参数解析-split_commandline函数
date: 2023-06-01 12:39:36
tags:
- ffmpeg
---

**函数原型**
```
int split_commandline(OptionParseContext *octx, int argc, char *argv[],
                      const OptionDef *options,
                      const OptionGroupDef *groups, int nb_groups);
```
将命令行拆分成内部格式，方便后续处理。

命令行被假定为由两种选项组成，一种属于一个组（选项标记包含OPT_SPEC, OPT_OFFSET or OPT_PERFILE），一种属于全局选项。

组(由OptionGroupDef结构体定义)是由组分隔符选项(例如-i)或非选项(不以-开头)终止的选项序列。没有分隔符选项的组必须始终位于所提供的组列表的第一个。

同一组中的所有选项存储在OptionGroupList中的一个OptionGroup结构中，具有相同组定义的所有组存储在OptionParseContext.groups中的一个OptionGroupList中。组列表的顺序与组定义的顺序相同。

**调用**

![ffmpeg_parser_options](ffmpeg_parser_options.png)
第三个参数传递的是options，这是一个预定义的OptionDef选项数组。

第四个参数groups定义如下
```
static const OptionGroupDef groups[] = {
    [GROUP_OUTFILE] = { "output url",  NULL, OPT_OUTPUT },
    [GROUP_INFILE]  = { "input url",   "i",  OPT_INPUT },
};
```
说明有两个选项组，一个包含输出参数，一个包含输入参数。
![选项解析涉及的结构体](选项解析涉及的结构体.png)

下面对函数源码进行简单的注释
![split_commandline-1](split_commandline-1.png)
![split_commandline-2](split_commandline-2.png)
![split_commandline-3](split_commandline-3.png)

