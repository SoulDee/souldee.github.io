---
title: Part 1：基础议题
urlname: bxf9xo
date: '2022-02-10 00:33:40 +0800'
categories:
  - Program
  - Cpp
  - More Effective Cpp
tags:
  - C++
  - 技巧
---

​

<!-- more -->

## 条款 01：仔细区别 pointers 和 references

指针和引用都是某个对象的别名，但他们又不完全一样，因此我们要根据具体情况来选择使用哪一个。

不存在 null 引用，一个引用必须总是代表某个对象，因此引用必须初始化，而指针可以为 null。

```cpp
int main()
{
   int counter = 0;
   int &r = counter;

   int *p = nullptr;
}
```

引用类型可以无需检测是否为 null 就使用，而指针需要检测

```cpp
void func(const MyClass &c) {
    // use c do something
}

void func(int *c) {
    if(MyClass) {
        // use c do something
    }
}
```

引用总是指向最初指向的哪个对象，而指针可以重新赋值。

```cpp
MyClass *p = nullptr;
p = new MyClass;
```

在一些情况也需要使用引用，例如 operator[]，这个操作符必须返回“能够被当作 assignment 赋值对象”的东西

基于以上原因得出结论，当你需要指向某个对象且不会改变指向，或实现一个操作符无法由指针达成就使用引用，否则使用指针。

## 条款 02: 最好使用 C++ 转型操作符

## 条款 03: 绝对不要以多态（polymorphically）方式处理数组

## 条款 04: 非必要不提供 default constructor
