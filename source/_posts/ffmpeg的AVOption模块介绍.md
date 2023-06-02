---
title: ffmpeg的AVOption模块介绍
date: 2023-06-01 10:18:18
tags:
- ffmpeg
---

AVOption提供一个通用系统，用于在任意结构体里声明选项。一个选项可以有帮忙文本，类型和可能的值范围。选项可以枚举，读写。

AVOption结构体如下：
```c
/**
 * AVOption
 */
typedef struct AVOption {
    const char *name;

    /**
     * short English help text
     * @todo What about other languages?
     */
    const char *help;

    /**
     * The offset relative to the context structure where the option
     * value is stored. It should be 0 for named constants.
     */
    int offset;
    enum AVOptionType type;

    /**
     * the default value for scalar options
     */
    union {
        int64_t i64;
        double dbl;
        const char *str;
        /* TODO those are unused now */
        AVRational q;
    } default_val;
    double min;                 ///< minimum valid value for the option
    double max;                 ///< maximum valid value for the option

    int flags;
    // 标记宏被我省略了...

    /**
     * The logical unit to which the option belongs. Non-constant
     * options and corresponding named constants share the same
     * unit. May be NULL.
     */
    const char *unit;
} AVOption;
```

AVClass结构体如下：
```c
/**
 * Describe the class of an AVClass context structure. That is an
 * arbitrary struct of which the first field is a pointer to an
 * AVClass struct (e.g. AVCodecContext, AVFormatContext etc.).
 */
typedef struct AVClass {
    /**
     * The name of the class; usually it is the same name as the
     * context structure type to which the AVClass is associated.
     */
    const char* class_name;

    /**
     * A pointer to a function which returns the name of a context
     * instance ctx associated with the class.
     */
    const char* (*item_name)(void* ctx);

    /**
     * a pointer to the first option specified in the class if any or NULL
     *
     * @see av_set_default_options()
     */
    const struct AVOption *option;

    /**
     * LIBAVUTIL_VERSION with which this structure was created.
     * This is used to allow fields to be added without requiring major
     * version bumps everywhere.
     */

    int version;

    /**
     * Offset in the structure where log_level_offset is stored.
     * 0 means there is no such variable
     */
    int log_level_offset_offset;

    /**
     * Offset in the structure where a pointer to the parent context for
     * logging is stored. For example a decoder could pass its AVCodecContext
     * to eval as such a parent context, which an av_log() implementation
     * could then leverage to display the parent context.
     * The offset can be NULL.
     */
    int parent_log_context_offset;

    /**
     * Category used for visualization (like color)
     * This is only set if the category is equal for all objects using this class.
     * available since version (51 << 16 | 56 << 8 | 100)
     */
    AVClassCategory category;

    /**
     * Callback to return the category.
     * available since version (51 << 16 | 59 << 8 | 100)
     */
    AVClassCategory (*get_category)(void* ctx);

    /**
     * Callback to return the supported/allowed ranges.
     * available since version (52.12)
     */
    int (*query_ranges)(struct AVOptionRanges **, void *obj, const char *key, int flags);

    /**
     * Return next AVOptions-enabled child or NULL
     */
    void* (*child_next)(void *obj, void *prev);

    /**
     * Iterate over the AVClasses corresponding to potential AVOptions-enabled
     * children.
     *
     * @param iter pointer to opaque iteration state. The caller must initialize
     *             *iter to NULL before the first call.
     * @return AVClass for the next AVOptions-enabled child or NULL if there are
     *         no more such children.
     *
     * @note The difference between child_next and this is that child_next
     *       iterates over _already existing_ objects, while child_class_iterate
     *       iterates over _all possible_ children.
     */
    const struct AVClass* (*child_class_iterate)(void **iter);
} AVClass;
```

- **AVOption实现**

这部分描述如何向一个结构体添加AVOption支持。

所有和AVOption相关的信息存储在一个叫AVClass的结构体中。因此结构体的第一个成员应该是指向AVClass的指针。AVClass的option成员必须指向一个静态的AVOption数组，数组以NULL结尾。

每个AVOption选项必须有一个非空的名字，一个类型，一个默认的值；对于值类型的AVOption也必须有一个允许的值范围；包含一个偏移量，指定结构体中和这个选项关联的字段的位置；AVOption中其他字段不是必须的，按情况赋值。

下面的例子展示了带AVOption支持的结构体：
```c
typedef struct test_struct {
  const AVClass *class;
  int      int_opt;
  char    *str_opt;
  uint8_t *bin_opt;
  int      bin_len;
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
  .class_name = "test class",
  .item_name  = av_default_item_name,
  .option     = test_options,
  .version    = LIBAVUTIL_VERSION_INT,
 };
```
当你分配结构体时，要确保AVClass指针指向正确的值，av_opt_set_defaults函数可以用来初始化。之后，该结构体就可以与AVOption API一起使用了。

继续上面的例子，我们展示结构体的构造和释放：
```c
test_struct *alloc_test_struct(void)
{
  test_struct *ret = av_mallocz(sizeof(*ret));
  ret->class = &test_class;
  av_opt_set_defaults(ret);
  return ret;
}

void free_test_struct(test_struct **foo)
{
  av_opt_free(*foo);
  av_freep(foo);
}
```

- **AVOption嵌套**

可能存在支持AVOption的结构体包含另一个支持AVOption的结构体

(比如AVCodecContext可以导出通用选项，然而它的priv_data字段可以导出codec相关的选项)。在这种情况下，可以设置父结构体导出一个子结构体的选项，为了实现这个功能，只要在父结构体的AVClass中实现AVClass.child_next()函数和
AVClass.child_class_iterate函数。

假设上面的test_struct结构体现在包含一个child_struct成员：
```c
typedef struct child_struct {
  AVClass *class;
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
```
将child_next和child_class_iterate放入test_class中：
```c
static const AVClass test_class = {
  .class_name          = "test class",
  .item_name           = av_default_item_name,
  .option              = test_options,
  .child_next          = child_next,
  .child_class_iterate = child_class_iterate,
  .version             = LIBAVUTIL_VERSION_INT,
 };
 ```
现在可以通过test_struct访问child_struct的选项。

从上面的例子中，可能不清楚为什么同时需要child_next和child_class_iterate，区别是child_next会遍历实际存在的对象，而child_class_iterate会遍历所有可能的子类。比如如果AVCodecContext被初始化为使用具有私有选项的编解码器，那么它的child_next会返回AVCodecContext.priv_data并结束遍历。而AVCodecContext.av_class中的child_class_iterate会遍历所有具有私有选项的可用编解码器。

- **命名常量**

可以为选项创建命名常量。

只需设置选项的unit字段，并将常量本身创建为AV_OPT_TYPE_CONST类型的选项，其unit字段设置为相同的字符串。它们的default_val字段应该包含命名常量的值。

例如，要为上面的test_flags选项添加一些命名常量，请将以下内容放入child_opts数组中：
```c
{ "test_flags", "This is a test option of flags type.",
  offsetof(child_struct, flags_opt), AV_OPT_TYPE_FLAGS, { .i64 = 0 }, INT_MIN, INT_MAX, .unit="test_unit" },
{ "flag1", "This is a flag with value 16", 0, AV_OPT_TYPE_CONST, { .i64 = 16 }, 0, 0, .unit="test_unit" },
```

- **AVOption使用**

本节讨论如何在支持AVOption的结构体中访问选项。支持AVOption的结构体有libavcodec模块的AVCodecContext以及libavformat模块的AVFormatContext等。

1. 检查AVOption

检查选项的基本函数是av_opt_next()和av_opt_find()，前者遍历一个对象定义的所有选项，后者搜索具有给定名称的选项。

嵌套的情况会更复杂一些。一个支持AVOption的结构体可能包含另一个支持AVOption的结构体。将AV_OPT_SEARCH_CHILDREN标志传递给av_opt_find()将使函数递归地搜索包含的结构体。

对于枚举，基本上有两种情况。第一种情况是当你想要获得结构及其子结构上可能存在的所有选项时(例如在构造文档时)。在这种情况下，你应该在父结构的AVClass上递归调用
av_opt_child_class_iterate()。第二种情况是，当你有一个已经初始化的结构，你想要从它获得所有可以实际写入或读取的选项。在这种情况下，您应该递归调用av_opt_child_next()，并在结果上调用av_opt_next()。

1. 读写AVOption

在设置选项时，通常会直接从用户处读取字符串。在这种情况下，简单地将其传递给av_opt_set()就足够了。对于非字符串类型的选项，av_opt_set()将根据选项类型解析字符串。

类似地，av_opt_get()将读取任何选项类型并将其转换为字符串并返回。别忘了字符串是已分配的，所以你必须用av_free()释放它。

在某些情况下，将所有选项放入AVDictionary并在其上调用av_opt_set_dict()可能更方便。

一个具体的例子是lavf/lavc中的format/codec open函数，它以一个填充options的字典作为参数。




