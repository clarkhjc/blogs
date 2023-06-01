---
title: C++对象模型(4)-vtable和vptr的类型问题
date: 2023-06-01 22:07:03
tags:
- C++
---

我们知道vtable是一个函数指针数组，存放的是虚函数地址，但是从语言层面讲，只有相同类型的对象才能放入同一个数组中，也就是说，你无法声明一个可以同时包含多种不同类型对象的数组，对函数指针数组来说也是如此。

其实C++编译器定义了一个通用的函数指针类型：
```c++
typedef void(*Pvfn)(void); // 通用的虚函数指针类型
```

vtable类型如下（伪代码）：
```c++
typedef struct {
    Typeinfo *_pinfo;
    Pvfn _array[];  // 虚函数个数由初始化语句确定 
} Vtable;
```

在每一个继承分支中的第一个多态类中插入vptr，而在每一个多态类中都插入vtable的声明
举例如下（伪代码）：
```c++
class Shape {
    Pvfn *_vptr;
    static Vtable _vtable;
 public:
    Shape(): m_color(0){}
    virtual ~Shape(){}
    float GetColor() const {return m_color;}
    void SetColor(float color) {m_color = color;}
    virtual void Draw() = 0;
private:
    float m_color;
};

class Rectangle : public Shape {
    static Vtable _vtable; // 在实现文件中初始化
public:
    Rectangle(): m_length(1), m_width(1) {}
    ~Rectangle() {}
    virtual void Draw() {}
......
}
```

编译器在编译完一个class后，把class中所有虚函数的指针（实际就是被转换为全局函数后的地址，函数地址是编译时常量）都强制转换为Pvfn类型，并用它们初始化_vtable的_array数组；而_vptr则被初始化为指向_vtable对象，具体指向哪里取决于编译器实现，比如g++编译器会指向vtable中第一个虚函数指针，可以参见C++对象模型(3)-使用gdb查看vtable。

在遇到通过指针或者引用调用虚函数的语句时，首先根据指针或者引用的静态类型来判断所调虚函数是否属于该class或者它的某个public基类，然后进行静态类型检查，例如：
```c++
Shape *pShape = new Rectangle;
pShape->Draw(); // 根据Shape::Draw()执行类型检查
```

检查通过后，就剩下最后一步工作了：改写虚函数调用语句，改写类似下面的样子：
```c++
(*(pShape->_vptr[2]))(pShape); //pShape->Draw();
```
具体索引值取决于vtable的构造。

但是，语句(pShape->_vptr[n])从vtable中取出来的函数指针类型应该是Pvfn，与实际调用的虚函数的类型一般是不匹配的（编译器知道我们定义的每一个虚函数的类型），所以还应该有一个反向类型强制转换的过程。最后的结果应该类似下面这样（仅作示意）：
```c++
typedef void(*Pvfn_Draw)(void);
(*(Pvfn_Draw)(pShape->_vptr[2]))(pShape);//pShape->Draw();  
```

在整个过程中并没有派生类改写的虚函数Rectangle::Draw()参与其中。那怎么会调用到这个改写的虚函数呢？奥妙就在vtable的构造及pShape当前实际指向的对象。

虽然pShape的静态类型是Shape*，但是在运行时它却指向一个Rectangle对象，而该对象的vptr成员指向Rectangle::_vtable，而不是Shape::_vtable；这个vtable中存放的也都是Rectangle改写过的虚函数或者新加的虚函数的地址，而不是Shape的虚函数地址，除非有的虚函数Rectangle没有改写。
