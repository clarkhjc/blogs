---
title: ffmpeg命令行参数解析-结构体
tags:
  - ffmpeg
abbrlink: 2c468ba2
date: 2023-06-01 12:52:47
---

ffmpeg命令行功能强大，本文简单介绍一下命令行解析过程。
命令行解析函数如下:
![ffmpeg_parse_options](ffmpeg_parse_options.png)

我们以下面命令行为例:
```
ffmpeg -y -ss 4 -i 1.ts -vframes 1 -f image2 -s 640x360 out.jpg
```
在ffmpeg_opt.c文件中通过两个函数split_commandline、parse_optgroup解析命令行参数并保存在OptionParseContext结构体中。
```c
typedef struct OptionParseContext {
    OptionGroup global_opts;
    OptionGroupList *groups;
    int           nb_groups;
    /* parsing state */
    OptionGroup cur_group;
} OptionParseContext;
```

OptionParseContext中global_opts保存全局参数，比如-y参数。

nb_groups表示有多少个OptionGroupList选项组列表，数据保存在groups数组中。

按输入参数还是输出参数进行分组，groups[0]保存输出参数选项组列表，

groups[1]保存输入参数选项组列表。

![vscode-debug-1](vscode-debug-1.jpeg)

```c
typedef struct OptionGroupList {
    const OptionGroupDef *group_def;
    OptionGroup *groups;
    int       nb_groups;
} OptionGroupList;
```
选项组列表保存一系列选项组，选项组个数保存在nb_groups字段，groups是选项组数组首地址。

从上面调试截图可以看出针对举例的命令行，输出和输入选项组都只有一个。
```c
typedef struct OptionGroup {
    const OptionGroupDef *group_def;
    const char *arg;

    Option *opts;
    int  nb_opts;

    AVDictionary *codec_opts;
    AVDictionary *format_opts;
    AVDictionary *sws_dict;
    AVDictionary *swr_opts;
} OptionGroup;
```

选项组包含若干选项，每一个选项都是保存在Option结构体中。
![vscode-debug-2](vscode-debug-2.jpeg)

输出选项组包含3个选项，
![vscode-debug-3](vscode-debug-3.jpeg)

从key/value值可以看出和我们在命令行里设置的输出参数一样， -vframes 1 -f image2 -s 640x360。
![vscode-debug-4](vscode-debug-4.jpeg)

输出选项组包含1个选项，
![vscode-debug-5](vscode-debug-5.png)

从key/value值可以看出和我们在命令行里设置的输入参数一样，-ss 4。
```c
typedef struct Option {
    const OptionDef  *opt;
    const char       *key;
    const char       *val;
} Option;
```

```c
typedef struct OptionDef {
    const char *name;
    int flags;
#define HAS_ARG    0x0001
#define OPT_BOOL   0x0002
#define OPT_EXPERT 0x0004
#define OPT_STRING 0x0008
#define OPT_VIDEO  0x0010
#define OPT_AUDIO  0x0020
#define OPT_INT    0x0080
#define OPT_FLOAT  0x0100
#define OPT_SUBTITLE 0x0200
#define OPT_INT64  0x0400
#define OPT_EXIT   0x0800
#define OPT_DATA   0x1000
#define OPT_PERFILE  0x2000     /* the option is per-file (currently ffmpeg-only).
                                   implied by OPT_OFFSET or OPT_SPEC */
#define OPT_OFFSET 0x4000       /* option is specified as an offset in a passed optctx */
#define OPT_SPEC   0x8000       /* option is to be stored in an array of SpecifierOpt.
                                   Implies OPT_OFFSET. Next element after the offset is
                                   an int containing element count in the array. */
#define OPT_TIME  0x10000
#define OPT_DOUBLE 0x20000
#define OPT_INPUT  0x40000
#define OPT_OUTPUT 0x80000
     union {
        void *dst_ptr;
        int (*func_arg)(void *, const char *, const char *);
        size_t off;
    } u;
    const char *help;
    const char *argname;
} OptionDef;
```
选项主要数据保存在OptionDef结构体中。name是选项名，flags是选项标志（比如选项值类型、是否包含参数、输入参数还是输出参数、视频参数还是音频参数还是字幕参数等），联合体u中可以包含三种数据之一，

|  数据类型  | 说明                         |
|----------|------------------------------|
| dst_ptr  | 全局控制变量的地址 |
| func_arg | 选项对应的处理函数 |
| off      | OptionsContext结构体中对应的字段 |





