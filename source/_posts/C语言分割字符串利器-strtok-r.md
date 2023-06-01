---
title: C语言分割字符串利器-strtok_r
date: 2023-06-01 22:11:14
tags:
- C
---

strtok_r函数可以根据指定的分隔符来切分字符串。
函数原型如下：
```c
#include <string.h>
char *strtok_r(char *str, const char *delim, char **saveptr);
```
str：需要分割的字符串
delim：分隔符
saveptr：保存剩下待分割的字符串。

比如需要将"Fred male 25,John male 62,Anna female 16"这个字符串按逗号分隔，代码如下：
```c
#include <stdio.h>
#include <string.h>

/*
 char *strtok_r(char *str, const char *delim, char **saveptr);
 注意：
 1、第一次调用时，str传递需要分割的字符串; 后续调用时，str需要传递NULL
 2、这个函数会修改str字符串，所以str不能是常量字符串
 3、返回的结果不包含delim字符
 */

int main(void)
{
    char buffer[100] = "Fred male 25,John male 62,Anna female 16";
    char *str = buffer;
    char *saveptr = NULL;
    char *ret = NULL;
    
    while ((ret = strtok_r(str, ",", &saveptr)) != NULL)
    {
        str = NULL;
        printf("ret: %s\n", ret);
    }
    return 0;
}
```
终端打印：
![strtok测试打印](strtok.png)

while循环了3次，每次输出一个字符串。

⚠️这个函数有几点需要注意：

1. 第一次调用时，str传递需要分割的字符串; 后续调用时，str需要传递NULL。

2. 这个函数会修改str字符串，所以str不能是常量字符串。

3. 返回的结果不包含delim字符。
