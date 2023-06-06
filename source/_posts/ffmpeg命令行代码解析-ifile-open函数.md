---
title: ffmpeg命令行代码解析-ifile_open函数
tags:
  - ffmpeg-src
abbrlink: c23817ad
date: 2023-06-01 09:49:45
---
ifile_open函数主要实现打开一个文件，获取封装和解码信息，seek源文件等功能。

主要调用函数包括avformat_alloc_context、avformat_open_input、

avformat_find_stream_info、avformat_seek_file等。

信息保存在Demuxer结构体里，

```c
typedef struct Demuxer {
    InputFile f;

    // name used for logging
    char log_name[32];

    /* number of times input stream should be looped */
    int loop;
    /* actual duration of the longest stream in a file at the moment when
     * looping happens */
    int64_t duration;
    /* time base of the duration */
    AVRational time_base;

    /* number of streams that the user was warned of */
    int nb_streams_warn;

    AVThreadMessageQueue *in_thread_queue;
    int                   thread_queue_size;
    pthread_t             thread;
    int                   non_blocking;
} Demuxer;
```

Demuxer主要成员是InputFile结构体

```c
typedef struct InputFile {
    const AVClass *class;

    int index;

    AVFormatContext *ctx;
    int eof_reached;      /* true if eof reached */
    int eagain;           /* true if last read attempt returned EAGAIN */
    int64_t input_ts_offset;
    int input_sync_ref;
    /**
     * Effective format start time based on enabled streams.
     */
    int64_t start_time_effective;
    int64_t ts_offset;
    /**
     * Extra timestamp offset added by discontinuity handling.
     */
    int64_t ts_offset_discont;
    int64_t last_ts;
    int64_t start_time;   /* user-specified start time in AV_TIME_BASE or AV_NOPTS_VALUE */
    int64_t recording_time;

    /* streams that ffmpeg is aware of;
     * there may be extra streams in ctx that are not mapped to an InputStream
     * if new streams appear dynamically during demuxing */
    InputStream **streams;
    int        nb_streams;

    int rate_emu;
    float readrate;
    int accurate_seek;

    /* when looping the input file, this queue is used by decoders to report
     * the last frame duration back to the demuxer thread */
    AVThreadMessageQueue *audio_duration_queue;
    int                   audio_duration_queue_size;
} InputFile;
```

InputFile包含封装层上下文结构体AVFormatContext、记录时间戳的多个变量、文件中每条流的信息，音视频流信息保存在InputStream数组中。

函数的大致流程图如下：

![ifile_open函数流程](ifile_open函数流程.png)

在seek源文件的时候有两个选项比较重要，一个是-seek_timestamp、一个是-ss，两者如何影响seek见下图：

![演示seek_timestamp和ss选项用法](演示seek_timestamp和ss选项用法.png)

seek_timestamp默认值是0，所以在只指定-ss情况下，会累加上文件的开始时间戳。
