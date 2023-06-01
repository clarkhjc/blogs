---
title: ffmpeg命令行参数解析-write_option函数
date: 2023-06-01 11:57:13
tags:
- ffmpeg
---

通过函数split_commandline解析命令行参数并保存在OptionParseContext结构体中。

然后通过函数parse_optgroup将OptionParseContext结构体中的数据解析到OptionsContext结构体中。

OptionsContext结构体中各个字段是最终控制ffmpeg行为的。

parse_optgroup代码如下：
![parse_optgroup](parse_optgroup.png)

函数循环遍历OptionGroup中的每个Option，并调用write_option将结果写到OptionsContext对应的结构体成员中。

write_option有点抽象，我们重点分析下，
![write_option](write_option.png)

我们先看函数的前半部分，是解析包含OPT_SPEC标记选项的，具有OPT_SPEC标记的选项会在OptionsContext结构体中对应一个SpecifierOpt指针成员以及一个表示SpecifierOpt大小的int型变量，比如:
![OptionsContext](OptionsContext.jpeg)

我们以codec_names成员的赋值为例：

codec_names是SpecifierOpt结构体指针变量，它会指向一个或者多个SpecifierOpt变量，相当于SpecifierOpt数组；紧接着的变量nb_codec_names表示数组的大小。

SpecifierOpt结构体如下：
![SpecifierOpt](SpecifierOpt.jpeg)

从预定义的OptionDef数组options可以得知，命令行中出现-c / -c:v / -c:a / -codec时，都会将后面的值保存在codec_names中。
![options](options.png)

`.off = OFFSET(codec_names)` 在选项定义的时候会计算出codec_names成员在OptionsContext结构体中的偏移量，在write_option中根据结构体首地址以及成员偏移量可以得到成员的访问地址:
```
void *dst = po->flags & (OPT_OFFSET | OPT_SPEC) ?
                (uint8_t *)optctx + po->u.off : po->u.dst_ptr;
```
codec_names是SpecifierOpt指针变量，dst指向的是codec_names的访问地址，所以dst相当于是指向SpecifierOpt指针的指针
```
SpecifierOpt **so = dst;
```
![OptionsContext_1](OptionsContext_1.png)
optctx是ptr0，dst是ptr5。

```
dstcount = (int *)(so + 1);
```
(so + 1)相当于是(dst + sizeof(SpecifierOpt *))，得到的是ptr6，即nb_codec_names变量的访问地址。
```
*so = grow_array(*so, sizeof(**so), dstcount, *dstcount + 1);
```
通过grow_array向codec_names数组中添加一个SpecifierOpt变量，并把nb_codec_names加1。
```
(*so)[*dstcount - 1].specifier = str;
```
将-c或者-c:后面的值赋值给新添加的SpecifierOpt变量的specifier字段.
```
dst = &(*so)[*dstcount - 1].u;
```
将SpecifierOpt的union成员地址赋值给dst。
```
if (po->flags & OPT_STRING) {
        // ...
        *(char **)dst = str;
}
```
如果选项后跟一个string类型的值，将string值赋值给union成员中的str字段。

以下列命令行为例：
```
ffmpeg -i input.mp4 -c:v mpeg4 -c:a aac output.mp4
```

![open_files](open_files.png)
parse_optgroup执行完成后OptionsContext中nb_codec_names和codec_names的值如上图所示。




