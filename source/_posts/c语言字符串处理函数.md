---
title: c语言字符串处理函数strspn和strcspn
tags:
  - C
abbrlink: '56528446'
date: 2023-06-20 09:58:14
---

# strspn -计算字符串开头和子串重叠的字符个数
从字符串的第一个字符开始，字符串和子串对应位置进行比较，遇到不相同的，即结束比较。
函数原型:
```c
size_t strspn(const char *s, const char *accept);
```
示意:
s0 s1 s2 s3 s4 ...
a0 a1 a2 a3 a4 ...
s0和a0比较，若相等，则比较s1和a1，若相等，则比较s2和a2...,最后返回相等的个数。

```c
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) {
    size_t ret1 = strspn("123456", "1");
    size_t ret2 = strspn("123456", "12");
    size_t ret3 = strspn("123456", "123");
    size_t ret4 = strspn("123456", "234");
    fprintf(stderr, "%ld %ld %ld %ld\n", ret1, ret2, ret3, ret4);

    return 0;
}
```
输出为：
```
1 2 3 0
```

# strcspn - 计算字符串开头不在子串中的字符的个数
从字符串的第一个字符开始，如果字符不在子串中，则接着比较下一个字符，如果在子串中，则结束比较。
函数原型:
```c
size_t strcspn(const char *s, const char *reject);
```
示意:
s0 s1 s2 s3 s4 ...
a0 a1 a2 a3 a4 ...
判断s0是否在a0-an中，如果不在，则判断s1是否在a0-an中，如果在则结束比较，返回不在的字符的个数。

```c
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) {
    size_t ret1 = strcspn("123456", "1");
    size_t ret2 = strcspn("123456", "2");
    size_t ret3 = strcspn("123456", "3");
    size_t ret4 = strcspn("123456", "34");
    fprintf(stderr, "%ld %ld %ld %ld\n", ret1, ret2, ret3, ret4);

    return 0;
}
```
输出为：
```
0 1 2 2
```

可以利用strcspn判断字符串是否以某些特殊字符开头，比如想判断字符串`s`是否以`#\r\n`三个字符中的某一个开头：
```c
#include <stdio.h>
#include <string.h>

int main() {
    const char *s = "#hello"; 
    if (strcspn(s, "#\r\n") == 0) {
        fprintf(stderr, "yes\n");
    } else {
        fprintf(stderr, "no\n");
    }

    const char *s1 = "\rworld";
    if (strcspn(s, "#\r\n") == 0) {
        fprintf(stderr, "yes\n");
    } else {
        fprintf(stderr, "no\n");
    }
    return 0;
}
```
输出：
```
yes
yes
```