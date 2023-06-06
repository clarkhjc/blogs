---
title: C语言函数指针
tags:
  - C
abbrlink: 5de992f2
date: 2023-06-03 13:56:34
---

# 函数指针
函数指针，顾名思义，就是指向函数的指针，它本质是一个指针，指向一个函数的首地址。

- 函数指针定义:

```c
int (*p)(int, int);
```
定义一个函数指针变量p，p指向的函数返回值是int，形参是两个int。

- 函数指针使用

```c
#include <stdio.h>
int add(int a, int b)
{
    return a + b;
}
int main() {
    // 函数名add加不加&都可以
    int (*p)(int, int) = &add;
    // 或者
    /*
    int (*p)(int, int)；
    p = add;
    */
    
    int sum = (*p)(2, 3);
    printf("sum = %d\n", sum);
    return 0;
}
```

函数先声明，后定义也可以使用:
```c
#include <stdio.h>

int add(int a, int b);

int main()
{
    int (*p)(int, int);
    p = add;
    
    int sum = (*p)(2, 3);
    
    printf("sum = %d\n", sum);
    return 0;
}

int add(int a, int b)
{
    return a + b;
}
```

# 函数指针数组

- 函数指针数组定义:
  
```c
int (*p[5])(int x, int y);
```
p先和[]结合成为一个数组，该数组内存储了五个指向函数的指针，所指向函数有两个int参数，并且返回值类型也是int。

- 函数指针数组使用

```c
#include <stdio.h>

int add(int a, int b)
{
    return a + b;
}

int sub(int a, int b)
{
    return a - b;
}

int main()
{
    int (*p[2])(int, int);
    p[0] = add;
    p[1] = sub;
    
    int sum = (*p[0])(2, 3);
    int sub = (*p[1])(3, 2);
    
    printf("sum = %d\n", sum);
    printf("sub = %d\n", sub);
    return 0;
}
```

>http://c.biancheng.net/view/228.html
>https://blog.csdn.net/X_Perseverance/article/details/79512908
>https://blog.csdn.net/fanshunguo/article/details/121100305





