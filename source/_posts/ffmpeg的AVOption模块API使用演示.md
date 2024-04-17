---
title: ffmpeg的AVOption模块API使用演示
tags:
  - ffmpeg
abbrlink: 4338da24
date: 2023-06-01 10:13:15
---

- **查找选项**

1. av_opt_next/av_opt_find函数可以直接在AVClass对象上查找选项：
```c
// 1、API直接操作AVClass结构体

// 通过av_opt_next查找AVClass中的所有选项
const AVClass *test_c = &test_class;
const AVOption *ret_c = NULL;
while ((ret_c = av_opt_next(&test_c, ret_c))) {
    printf("av_opt_next->AVClass option name: %s\n", ret_c->name);
}
printf("\n");

// 通过av_opt_find查找AVClass中的某个选项 包括子结构体中的选项
// serach_flags需要设置为AV_OPT_SEARCH_CHILDREN | AV_OPT_SEARCH_FAKE_OBJ
const AVOption *find_c = av_opt_find(&test_c, "test_flags", NULL, 0, AV_OPT_SEARCH_CHILDREN | AV_OPT_SEARCH_FAKE_OBJ);
if (find_c) {
    printf("av_opt_find->AVClass option name: %s\n", find_c->name);
}
printf("\n");
```

2. 查找结构体中子结构体的AVClass并遍历子结构体的所有选项：
```c
// 查找结构体中子结构体的AVClass并遍历子结构体的所有选项
void *iter = NULL;
const AVClass *child_c = av_opt_child_class_iterate(test_c, &iter);
if (child_c) {
    const AVOption *ret_c = NULL;
    while ((ret_c = av_opt_next(&child_c, ret_c))) {
        printf("av_opt_child_class_iterate->av_opt_next->AVClass option name: %s\n", ret_c->name);
    }
}
printf("\n");
```

3. API操作结构体对象
```c
test_struct *obj = alloc_test_struct();

// 演示av_opt_child_next和av_opt_next用法
void *child = NULL;
child = av_opt_child_next(obj, child);
if (child) {
    child_struct *child_obj = (child_struct *)child;

    const AVOption *ret = NULL;
    while ((ret = av_opt_next(child_obj, ret))) {
        printf("av_opt_child_next->av_opt_next->obj option name: %s\n", ret->name);
    }
}
printf("\n");

// 演示av_opt_find用法
const AVOption *find = av_opt_find(obj, "test_flags", NULL, 0, AV_OPT_SEARCH_CHILDREN);
if (find) {
    printf("av_opt_find->obj option name: %s\n", find->name);
}
printf("\n");
```

- **读写选项**
通过av_opt_get获取某个选项的值，通过av_opt_set设置某个选项的值：

```c
/*************************
 * 二、读写选项
 * **********************/
int res = -1;
uint8_t *buf = NULL;
res = av_opt_get(obj, "test_int", AV_OPT_SEARCH_CHILDREN, &buf);
if (res >= 0) {
    printf("before test_int value: %s\n", buf);
    av_free(buf);
}
res = av_opt_set(obj, "test_int", "100", AV_OPT_SEARCH_CHILDREN);
res = av_opt_get(obj, "test_int", AV_OPT_SEARCH_CHILDREN, &buf);
if (res >= 0) {
    printf("after test_int value: %s\n", buf);
    av_free(buf);
}
```

完整演示代码：
```c
#include <stdio.h>
#include <libavutil/opt.h>

void *child_next(void *obj, void *prev);
const AVClass* child_class_iterate(void **iter);

typedef struct child_struct {
  const AVClass *class;
  int flags_opt;
} child_struct;

static const AVOption child_opts[] = {
  { "test_flags", "This is a test option of flags type.",
   offsetof(child_struct, flags_opt), AV_OPT_TYPE_FLAGS, { .i64 = 0 }, INT_MIN, INT_MAX },
  { NULL },
};

static const AVClass child_class = {
  .class_name = "child class",
  .item_name  = av_default_item_name,
  .option     = child_opts,
  .version    = LIBAVUTIL_VERSION_INT,
};

typedef struct test_struct {
  const AVClass *class;
  int      int_opt;
  char    *str_opt;
  uint8_t *bin_opt;
  int      bin_len;
  struct child_struct *child_struct;
} test_struct;

static const AVOption test_options[] = {
  {"test_int", "This is a test option of int type.", offsetof(test_struct, int_opt),
    AV_OPT_TYPE_INT, { .i64 = -1 }, INT_MIN, INT_MAX},
  {"test_str", "This is a test option of string type.", offsetof(test_struct, str_opt),
    AV_OPT_TYPE_STRING },
  {"test_bin", "This is a test option of binary type.", offsetof(test_struct, bin_opt),
  AV_OPT_TYPE_BINARY},
  {NULL},
};

static const AVClass test_class = {
  .class_name          = "test class",
  .item_name           = av_default_item_name,
  .option              = test_options,
  .child_next          = child_next,
  .child_class_iterate = child_class_iterate,
  .version             = LIBAVUTIL_VERSION_INT,
 };

void *child_next(void *obj, void *prev)
{
  test_struct *t = obj;
  if (!prev && t->child_struct)
    return t->child_struct;
  return NULL;
}

const AVClass* child_class_iterate(void **iter)
{
  const AVClass *c = *iter ? NULL : &child_class;
  *iter = (void*)(uintptr_t)c;
  return c;
}

test_struct *alloc_test_struct(void)
{
  test_struct *ret = av_mallocz(sizeof(*ret));
  ret->class = &test_class;
  ret->child_struct = av_mallocz(sizeof(*ret->child_struct));
  ret->child_struct->class = &child_class;
  av_opt_set_defaults(ret->child_struct);
  av_opt_set_defaults(ret);
  return ret;
}

void free_test_struct(test_struct **foo)
{
  av_opt_free((*foo)->child_struct);
  av_opt_free(*foo);

  av_freep(&((*foo)->child_struct));
  av_freep(foo);
}

int main()
{
    /*************************
     * 一、查找选项
     * **********************/
    // 1、API直接操作AVClass结构体

    // 通过av_opt_next查找AVClass中的所有选项
    const AVClass *test_c = &test_class;
    const AVOption *ret_c = NULL;
    while ((ret_c = av_opt_next(&test_c, ret_c))) {
        printf("av_opt_next->AVClass option name: %s\n", ret_c->name);
    }
    printf("\n");

    // 通过av_opt_find查找AVClass中的某个选项 包括子结构体中的选项
    // serach_flags需要设置为AV_OPT_SEARCH_CHILDREN | AV_OPT_SEARCH_FAKE_OBJ
    const AVOption *find_c = av_opt_find(&test_c, "test_flags", NULL, 0, AV_OPT_SEARCH_CHILDREN | AV_OPT_SEARCH_FAKE_OBJ);
    if (find_c) {
        printf("av_opt_find->AVClass option name: %s\n", find_c->name);
    }
    printf("\n");

    // 查找结构体中子结构体的AVClass并遍历子结构体的所有选项
    void *iter = NULL;
    const AVClass *child_c = av_opt_child_class_iterate(test_c, &iter);
    if (child_c) {
        const AVOption *ret_c = NULL;
        while ((ret_c = av_opt_next(&child_c, ret_c))) {
            printf("av_opt_child_class_iterate->av_opt_next->AVClass option name: %s\n", ret_c->name);
        }
    }
    printf("\n");

    // 2、API操作结构体对象
    test_struct *obj = alloc_test_struct();

    // 演示av_opt_child_next和av_opt_next用法
    void *child = NULL;
    child = av_opt_child_next(obj, child);
    if (child) {
        child_struct *child_obj = (child_struct *)child;

        const AVOption *ret = NULL;
        while ((ret = av_opt_next(child_obj, ret))) {
            printf("av_opt_child_next->av_opt_next->obj option name: %s\n", ret->name);
        }
    }
    printf("\n");

    // 演示av_opt_find用法
    const AVOption *find = av_opt_find(obj, "test_flags", NULL, 0, AV_OPT_SEARCH_CHILDREN);
    if (find) {
        printf("av_opt_find->obj option name: %s\n", find->name);
    }
    printf("\n");

    /*************************
     * 二、读写选项
     * **********************/
    int res = -1;
    uint8_t *buf = NULL;
    res = av_opt_get(obj, "test_int", AV_OPT_SEARCH_CHILDREN, &buf);
    if (res >= 0) {
        printf("before test_int value: %s\n", buf);
        av_free(buf);
    }
    res = av_opt_set(obj, "test_int", "100", AV_OPT_SEARCH_CHILDREN);
    res = av_opt_get(obj, "test_int", AV_OPT_SEARCH_CHILDREN, &buf);
    if (res >= 0) {
        printf("after test_int value: %s\n", buf);
        av_free(buf);
    }
    free_test_struct(&obj);

    return 0;
}
```

打印输出：
![演示代码打印](演示代码打印.png)



