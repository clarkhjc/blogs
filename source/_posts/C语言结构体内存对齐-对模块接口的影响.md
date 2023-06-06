---
title: C语言结构体内存对齐--对模块接口的影响
tags:
  - C
abbrlink: 8aff2b28
date: 2023-06-01 22:18:06
---

对齐方式会影响到模块之间二进制兼容性。

在一个多模块的应用程序中，可能包含一个可执行程序、若干个静态链接库和若干个动态链接库。模块之间除了函数接口外，还会有一些共享复合数据类型定义，如果各个模块对齐方式不一致，会导致程序出错甚至崩溃。

举例，有一个应用程序exe链接一个静态库PrintSedan.lib，动态库的接口如下
```c
// sedan.h
#ifndef _SEDAN_H_
#define _SEDAN_H_

enum Color {RED = 0x01, BLUE, GREEN, YELLOW, BLACK};
typedef unsigned char BYTE;

struct Sedan {
    bool   m_hasSkylight;
    Color  m_color;
    bool   m_isAutoShift;
    double m_price;
    BYTE   m_seatNum;
};

#ifdef __cplusplus
extern "C" {
#endif
    void __cdecl print_Sedan_1(const Sedan *p);
#ifdef __cplusplus
}
#endif

#endif
```
在应用程序和静态库中打印结构体Sedan的各个成员的值。
```c
// sedan.cpp
#include "sedan.h"
#include <iostream>

using namespace std;

#ifdef __cplusplus
extern "C" {
#endif

    void __cdecl print_Sedan_1(const Sedan *p)
    {
        cout << "print_Sedan_1() in library PrintSedan.lib:" << endl;
        cout << "Sedan size: " << sizeof(Sedan) << endl;
        cout << "// Sedan: \n{\n";
        cout << "\tm_hasSkylight=" << (p->m_hasSkylight ? "yes" : "no") << endl;
        cout << "\tm_color=" << (p->m_color) << endl;
        cout << "\tm_isAutoShift=" << (p->m_isAutoShift ? "yes" : "no") << endl;
        cout << "\tm_price=" << (p->m_price) << endl;
        cout << "\tm_seatNum=" << (int)(p->m_seatNum) << endl;
        cout << "}\n\n";

    }
#ifdef __cplusplus
}
#endif
```
sedan.h和sedan.cpp编译生成静态库PrintSedan.lib，在vs工程中修改结构体成员对齐为8字节：
![vs工程结构体对齐设置](1.png)

应用程序代码为：
```c
// main.cpp
#include "sedan.h"
#include <iostream>

using namespace std;

void __cdecl print_Sedan_2(const Sedan *p)
{
    cout << "print_Sedan_2() in main.cpp:" << endl;
    cout << "Sedan size: " << sizeof(Sedan) << endl;
    cout << "// Sedan: \n{\n";
    cout << "\tm_hasSkylight=" << (p->m_hasSkylight ? "yes" : "no") << endl;
    cout << "\tm_color=" << (p->m_color) << endl;
    cout << "\tm_isAutoShift=" << (p->m_isAutoShift ? "yes" : "no") << endl;
    cout << "\tm_price=" << (p->m_price) << endl;
    cout << "\tm_seatNum=" << (int)(p->m_seatNum) << endl;
    cout << "}\n\n";

}

int main()
{
    Sedan s;
    s.m_hasSkylight = true;
    s.m_color = GREEN;
    s.m_isAutoShift = false;
    s.m_price = 100000;
    s.m_seatNum = 4;

    print_Sedan_2(&s);
    print_Sedan_1(&s);

    return 0;
}
```
main.cpp编译为SedanTest.exe，需要指定sedan.h为依赖的头文件，PrintSedan.lib为依赖的静态库。在vs工程中修改结构体成员对齐为4字节：
![vs工程中修改结构体成员对齐](2.png)

运行SedanTest.exe得到：
![运行输出](3.png)

从打印可见，库中对齐设置为8字节，结构体Sedan大小为32字节，应用程序对齐设置为4字节，结构体大小为24字节，这就导致二进制不兼容，最终导致在库中打印结构体Sedan时按照32字节处理，输出的m_price和m_seatNum值错误。

能够100%保证对齐一致的方法就是直接在代码中使用编译器提供的方法指定每一个接口数据类型的对齐方式。

比如在windows下，用#pragma显示指定Sedan结构体8字节对齐
```c
#pragma pack(push, 8)
struct Sedan {
    bool   m_hasSkylight;
    Color  m_color;
    bool   m_isAutoShift;
    double m_price;
    BYTE   m_seatNum;
};
#pragma pack(pop)
```
重新编译静态库和应用程序，打印输出如下：
![运行输出-成功](4.png)


