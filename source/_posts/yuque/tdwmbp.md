---
title: Part 7：模板与泛型编程（暂略）
urlname: tdwmbp
date: '2022-01-24 12:00:07 +0800'
categories:
  - 编程
  - C++
  - Effective C++
tags:
  - C++
  - 技巧
---

因 template 相关知识用的比较少，这一部分看的迷迷糊糊的，暂略，等待重新学一下《C++ Primer》相关内容后再重新看这一部分。
​

<!-- more -->

​

### 条款 41：了解隐式接口和编译器多态

class 和 template 编程都支持接口和多态，但是侧重有所不同，class 侧重与显式接口和运行时多态，而 template 侧重于隐式接口和编译器多态。
​

**class 多态：**那些被声明为 virtual 的函数，在运行时才根据动态类型决定调用哪一个函数.

```cpp
class Base {
public:
    virtual void log() {
        std::cout << "base" << std::endl;
    };
    virtual void log(int n) {
        std::cout << "base" << std::endl;
    };
};
class Derived: public Base {
public:
    void log() override {
        std::cout << "derived" << std::endl;
    };
};

int main()
{
    Base *b = new Derived();
    b->log(); // 被派生类覆写了，调用的 Derived::log
    b->log(10); // 没有被覆写，调用 Base::log
}
```

**template 多态：**在编译期间编译器根据 template 的不同参数的调用进行具现化和函数重载

```cpp
class System {
public:
    void log() const { std::cout << "system" << std::endl; };
};
class DataBase {
public:
    void log() const { std::cout << "database" << std::endl; };
};

template <typename T>
void func(const T& c) {
    c.log();
}

int main()
{
    // 以下会让编译器生成对应的函数实现
    System s;
    DataBase d;
    func(s);
    func(d);
}
```

​

**class 显式接口：**由函数的签名构成，例如上面提到的 log 函数。
**template 隐式接口：**由有效表达式组成，即在 template 函数中，并不要求你必须某个接口，只要最终能让表达式成立即可。

```cpp
// size 函数可以是基类的
// w 可以不重载 operator>，只要返回一个对象，这个对象加 int 类型后能够调用 >
// w 可以不重载 operator!=
// 总之，只要最终表达式可以成立
template <typename T>
void doProcessing(T& w) {
    if(w.size() > 10 && w != someNastyWidget)
        ...
}
```

### 条款 42：了解 typename 的双重意义

typename 和 class 从意义上来说没有什么不同的，而 typename 暗示了 typename 不一定是个类型，更为贴切。
​

嵌套从属名称会导致解析困难，因此在 template 中要指涉一个嵌套从属名称要使用 typename 显式的写出来。

```cpp
template <typename C>
void print2nd(const C& container) {
    // error: C::const_iterator 默认会被认为不是一个类型，这就变成相乘
    C::const_iterator* x;

    // 使用 typename 明确说明这是嵌套从属名称，是一个类型
    typename C::const_iterator* x;
}
```

但不能在基类列或成员初值列以 typename 修饰（暂略）

### 条款 43：学习处理模版化基类内的名称

### 条款 44：将与参数无关的代码抽离 templates

### 条款 45：运用成员函数模版接受所有兼容类型

### 条款 46：需要类型转换时请为模板定义非成员函数

### 条款 47：请使用 traits classes 表现类型信息

### 条款 48：认识 template 元编程
