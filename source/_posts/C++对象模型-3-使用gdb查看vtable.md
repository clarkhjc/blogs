---
title: C++对象模型(3)-使用gdb查看vtable
tags:
  - C++
abbrlink: ecd0eb74
date: 2023-06-01 21:55:30
---

我们首先执行下面的代码:
```c++
#include <iostream>
using namespace std;

class NonVirtualClass {
public:
    void foo() {}
};

class VirtualClass {
public:
    virtual void foo() {}
};

int main() {
    cout << "Size of NonVirtualClass: " << sizeof(NonVirtualClass) << endl;
    cout << "Size of VirtualClass: " << sizeof(VirtualClass) << endl;
}
```
用g++编译并执行:
```
$g++ 1.cpp && ./a.out
Size of NonVirtualClass: 1
Size of VirtualClass: 8
```
从“C++对象模型(1)”可知构成对象本身的只有数据，NonVirtualClass和VirtualClass类都没有数据成员。
NonVirtualClass大小是1是因为C++类的大小不能为0（因为需要给对象分配内存，用以区分不同的对象）。
VirtualClass大小为8，是因为有一个隐藏的指针，指针指向vtable。

下面通过gdb调试来查看对象的内存布局，可以深入了解vtable的组织结构。
示例代码如下：
```c++
#include <iostream>

class Parent {
public:
    virtual void Foo() {}
    virtual void FooNotOverridden() {}
};

class Derived : public Parent {
public:
    void Foo() override {}
};

int main() {
    Parent p1, p2;
    Derived d1, d2;

    std::cout << "done" << std::endl;
}
```
1. 编译生成带调试信息的可执行程序

```
$g++ -g 2.cpp
```

2. 调试可执行程序

```
$gdb a.out
(gdb) # configure gdb
(gdb) set print asm-demangle on
(gdb) set print demangle on
(gdb) # set breakpoint at main
(gdb) b main
Breakpoint 1 at 0x11d5: file main.cpp, line 14.
(gdb) run
Starting program: /home/clarkh/c++/a.out
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, main () at main.cpp:14
14      int main() {
(gdb) n
15        Parent p1, p2;
(gdb) n
16        Derived d1, d2;
(gdb) n
18        std::cout << "done" << std::endl;
(gdb) # print p1, p2, d1, d2 - we'll talk about what the output means soon
(gdb) p p1
$1 = {_vptr.Parent = 0x555555557d50 <vtable for Parent+16>}
(gdb) p p2
$2 = {_vptr.Parent = 0x555555557d50 <vtable for Parent+16>}
(gdb) p d1
$3 = {<Parent> = {_vptr.Parent = 0x555555557d30 <vtable for Derived+16>}, <No data fields>}
(gdb) p d2
$4 = {<Parent> = {_vptr.Parent = 0x555555557d30 <vtable for Derived+16>}, <No data fields>}
```
从打印我们可以了解到：
- 即使类没有数据成员，对象也有一个隐藏的指针，指针指向vtable；
- p1和p2指向的vtable相同，d1和d2指向的vtable也相同，vtable是静态数据，每一个类有一份数据；
- d1和d2从Parent类继承了一个指针，指向Derived的vtable；
- vptr不是指向vtable首地址，而是指向首地址偏移16字节的位置。

下面我们用gdb查看vtable的内容，使用gdb的x命令，将内存打印到终端。Derived对象的vptr指向0x555555557d30地址，我们在此基础上减去16字节，从vtable的首地址0x555555557d20开始打印：
```
(gdb) x/200xb 0x555555557d20
0x555555557d20 <vtable for Derived>:    0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0x555555557d28 <vtable for Derived+8>:  0x60    0x7d    0x55    0x55    0x55    0x55    0x00    0x00
0x555555557d30 <vtable for Derived+16>: 0xe6    0x52    0x55    0x55    0x55    0x55    0x00    0x00
0x555555557d38 <vtable for Derived+24>: 0xd6    0x52    0x55    0x55    0x55    0x55    0x00    0x00
0x555555557d40 <vtable for Parent>:     0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
0x555555557d48 <vtable for Parent+8>:   0x78    0x7d    0x55    0x55    0x55    0x55    0x00    0x00
0x555555557d50 <vtable for Parent+16>:  0xc6    0x52    0x55    0x55    0x55    0x55    0x00    0x00
0x555555557d58 <vtable for Parent+24>:  0xd6    0x52    0x55    0x55    0x55    0x55    0x00    0x00
0x555555557d60 <typeinfo for Derived>:  0x70    0x7c    0xfa    0xf7    0xff    0x7f    0x00    0x00
0x555555557d68 <typeinfo for Derived+8>:        0x10    0x60    0x55    0x55    0x55    0x55    0x00    0x00
0x555555557d70 <typeinfo for Derived+16>:       0x78    0x7d    0x55    0x55    0x55    0x55    0x00    0x00
0x555555557d78 <typeinfo for Parent>:   0xe0    0x6f    0xfa    0xf7    0xff    0x7f    0x00    0x00
0x555555557d80 <typeinfo for Parent+8>: 0x20    0x60    0x55    0x55    0x55    0x55    0x00    0x00
```

下面是Derived的vtable布局：
| 地址 | 值 | 意义 |
|-----|----|-----|
| 0x555555557d20 | 0x0 | top_offset |
| 0x555555557d28 | 0x555555557d60 | 指向typeinfo for Derived | 
| 0x555555557d30 | 0x5555555552e6 | 指向Derived::Foo()，Derived的_vptr指向这儿 |
| 0x555555557d38 | 0x5555555552d6 | 指向Parent::FooNotOverridden() |

下面是Parent的vtable布局：
| 地址 | 值 | 意义 |
|-----|----|-----|
| 0x555555557d40 | 0x0 | top_offset |
| 0x555555557d48 | 0x555555557d78 | 指向typeinfo for Parent |
| 0x555555557d50 | 0x5555555552c6 | 指向Parent::Foo()，Parent的_vptr指向这儿 |
| 0x555555557d58 | 0x5555555552d6 | 指向Parent::FooNotOverridden() |

通过info symbol指令可以查看某个地址的符号，比如:
```
(gdb) info symbol 0x5555555552c6
Parent::Foo() in section .text of /home/clarkh/c++/a.out
(gdb) info symbol 0x5555555552d6
Parent::FooNotOverridden() in section .text of /home/clarkh/c++/a.out
(gdb) info symbol 0x5555555552e6
Derived::Foo() in section .text of /home/clarkh/c++/a.out
```

我们从0x5555555552c6打印出代码段的内容：
```
(gdb) x/100xb 0x5555555552c6
0x5555555552c6 <Parent::Foo()>: 0xf3    0x0f    0x1e    0xfa    0x55    0x48    0x89    0xe5
0x5555555552ce <Parent::Foo()+8>:       0x48    0x89    0x7d    0xf8    0x90    0x5d    0xc3    0x90
0x5555555552d6 <Parent::FooNotOverridden()>:    0xf3    0x0f    0x1e    0xfa    0x55    0x48    0x89    0xe5
0x5555555552de <Parent::FooNotOverridden()+8>:  0x48    0x89    0x7d    0xf8    0x90    0x5d    0xc3    0x90
0x5555555552e6 <Derived::Foo()>:        0xf3    0x0f    0x1e    0xfa    0x55    0x48    0x89    0xe5
0x5555555552ee <Derived::Foo()+8>:      0x48    0x89    0x7d    0xf8    0x90    0x5d    0xc3    0x00
```

从Derived和Parent的vtable布局，我们可以看出vtable函数指针数组存放的数据如下：
| vtable数组元素 | 值 |
|--------------|----|
| vtable[0] | 0x0（top_offset）|
| vtable[1] | 指向typeinfo |
| vtable[2] | 指向第一个虚函数地址 |
| ... | 指向第n个虚函数地址 |

