---
title: ffmpeg的AVDictionary模块介绍
tags:
  - ffmpeg-src
abbrlink: 59095f4
date: 2023-06-01 10:07:30
---

字典用来存储键-值对。

相关结构体定义：

```c
typedef struct AVDictionaryEntry {
    char *key;
    char *value;
} AVDictionaryEntry;

struct AVDictionary {
    int count;
    AVDictionaryEntry *elems;
};
typedef struct AVDictionary AVDictionary;
```

1. 创建一个字典
   
```c
AVDictionary *d = NULL; // "create" an empty dictionary
```

NULL可以作为一个空字典，在任何需要指向AVDictionary指针的地方使用。

2. 添加一个键-值对

```c
av_dict_set(&d, "foo", "bar", 0); // add an entry
```

3. 遍历字典

```c
AVDictionaryEntry *t = NULL;
while ((t = av_dict_iterate(d, t))) {
   <....>                        // iterate over all entries in d
}
```

4. 获取某个键-值对

```c
AVDictionaryEntry *t = av_dict_get(d, "foo", NULL, AV_DICT_MATCH_CASE);
```

5. 释放字典以及包含的内容

```c
av_dict_free(&d);
```

6. 例子

```c
#include <stdio.h>
#include <libavutil/dict.h>

int main()
{
    AVDictionary *d = NULL;
    av_dict_set(&d, "name", "clark", 0);
    av_dict_set(&d, "age", "18", 0);
    av_dict_set(&d, "sex", "man", 0);
    av_dict_set(&d, "job", "engineer", 0);

    const AVDictionaryEntry *t = NULL;
    while ((t = av_dict_iterate(d, t))) {
        printf("%s -> %s\n", t->key, t->value);
    }

    t = av_dict_get(d, "job", NULL, AV_DICT_MATCH_CASE);

    av_dict_free(&d);
    return 0;
}
```

输出：

![output](output.png)