---
title: Part 4：设计与声明
urlname: uern2q
date: '2022-01-24 00:00:04 +0800'
categories:
  - 编程
  - C++
  - Effective C++
tags:
  - C++
  - 技巧
---

本章节教我们如何去设计一个良好的类和接口，哪些函数又应当成为非成员版本。
​

<!-- more -->

​

### 条款 18：让接口容易被正确使用，不易被误用

让接口易用而不被误用的基本准则是：如果代码通过编译，那么它的行为就应当是符合使用者预期的，否则不应当通过编译。简而言之，就是降低使用者的使用负担，为了实现这样的目的我们需要针对使用者可能犯的错误做一些应对：
​

**限制参数类型：**对于参数类型相同，但实际用途不同的接口，容易顺序误用，可以通过外覆类型来对参数的传递进行限制

```cpp
class Date {
public:
    Date(int y, int m, int d);      // bad：可能因为国家习惯不同导致传参顺序不同，编译器并不会因此报错
    Date(Year y, Month m, Day d);   // good：强制使用者类型正确，顺序也因此正确了
};

Date date(Year(2022), Month(1), Day(24));
```

**限制参数值：**对于一些值是有限的类型应当限制值，但考虑到 enum 可能被当成 int 使用，因此通过函数来限制

```cpp
class Week {
public:
    static Week Monday() { return Week(1);}
    static Week Tuesday() { return Week(2);}
    // ...
private:
    explicit Week(int w); // 阻止生成其他值，只允许上述 static 方法提供值
};

Week week = Week::Monday();
```

​

**限制类的行为：**通过语法和编译器的帮助来限制一些用户意外的犯错，典型就是 const 语法，详细见 **条款 03**

**限制行为差异性：**尽量的让你的类型表现同内置类型相似，一致性能够降低用户学习新的接口的学习成本和使用成本。例如标准容器类型都会提供一个 size 函数成员来查看有多少对象。
​

**降低资源管理责任：**对于一些需要资源管理的接口，应当假设用户可能会忘记释放。例如对于返回指针的工厂函数，修改为返回智能指针。

### 条款 19：设计 class 犹如设计 type

本条款提供了一系列问题来供我们设计类的参考，以下为问题和对应的设计关键词：
​

- **新 type 的对象应当如何创建和销毁：**构造，析构，new，delete 的设计
- **对象的初始化和对象的赋值该有什么样的差别：**构造，赋值操作符设计，初始化和赋值差异
- **新 type 的对象如果 passed by value ，意味着什么：**构造函数设计
- **什么是新 type 的“合法值”：**值的约束，错误检测，异常处理
- **你的新 type 需要配合某个继承图系吗：**受到基类成员函数是否 virtual 的影响，自身成员函数是否 virtual 对派生类影响
- **你的新 type 需要什么样的转换：**explicit 构造，类型转换函数或者专有的非成员函数的类型转换函数
- **什么样的操作符和函数对此新 type 而言是合理的：**是否为成员函数
- **什么样的标准函数应该驳回：**编译器默认生成函数，见条款 06
- **谁该取用性 type 的成员：**决定成员和成员函数的 public、protected、praive，决定是否 friends
- **什么是新 type 的“未声明接口”：**对效率、异常安全和资源运用管理需要的约束条件
- **你的新 type 有多么一般化：**决定定义 class template 来生成 types 家族还是定义一个类
- **你真的需要一个新 type 吗：**考虑派生类

### 条款 20：宁以 pass-by-reference-to-const 替换 pass-by-value

默认下 C++ 以值传递参数到函数，并调用拷贝构造函数返回副本作为初始值，这会有两个问题。
​

**昂贵的操作：**类构造函数，基类构造函数，成员的构造函数以及函数结束时调用对应的析构函数，对此的解决方式是采用 const 引用传递参数，引用避免了多次的构造和析构，const 则保证了该参数不会在函数中修改而影响原来的值。

```cpp
void fn(Week w); 		// bad：多次构造、析构
void fn(const Week& w);	// good：更高效
```

**对象切割：**即派生类传递给参数为基类的函数，此时会调用基类的构造，在函数中将作为基类使用，调用的也将是基类的 virtual 成员函数。

```cpp
void func(Base o) {
  o.fn();  // 调用基类版本的 fn
};

void func(const Base& o) {
    o.fn(); // 调用派生类的 fn
};

int main()
{
    Derived d;
    func(d);
}

```

**但如果是内置类型，STL 的迭代器和函数对象，由于 C++ 编译器往往以指针实现引用，因此使用值传递更为高效。**

还需要注意，不应当因为自定类型比较小而采用值传递，因为它们可能指向一个体型大的类型，指向的也会被复制，另一个原因在于你无法保证这个类型在未来是否会被扩称为大的类型。

### 条款 21：必须返回对象时，别妄想返回其 reference

不要为了节省一个构造和析构而将本应该返回新的对象的函数修改为返回引用。
​

函数返回局部变量的引用或指针将导致未定义行为，因为该变量在函数结束后就销毁了。

```cpp
MyClass & func() {
    MyClass c;
    return c;
}

func(); // 返回一个被销毁内存的引用或指针
```

函数返回局部分配了动态内存的对象，可能导致资源泄漏，因为无法合理的对该对象进行 delete

```cpp
Rational w, x, y, z;
w = x * y * z; // 相当于 operator*(operator* (x, y), z), 内部的 x * y 返回的指针无法 delete
```

函数返回局部 static 对象的引用或指针会有多线程安全性问题（条款 04），并且无论指针还是引用，其实都是对同一个对象的别名，这意味着当你用这个函数获得的对象用来相等对比时总是得到 true 的结果。

### 条款 22：将成员变量声明为 private

**语法一致性：**用户将只能通过函数来访问数据，而无法直接访问，也不需要在使用的时候验证要不要函数调用的小括号
​

**封装性：**封装性隐藏了类的内部细节，不再需要关注实现细节，一切数据操作通过函数进行，只要保证声明不变，以后需要更换细节实现也无需对函数调用进行修改，
​

成员变量声明 pirvate，影响到的将只是该类本身的函数，声明为 public 的话，一旦修改了该成员变量的声明，每个用到的地方都要修改，而声明为 protected 也将会影响到派生类。

```cpp
class MyClass {
public:
    int fn() {
        // 算法 A
        int a = 10, b = 5;
        return a * b;

        // 新的算法 B，不会影响到用户
        int a = 10, b = 5;
        return a + b;
    };
};
```

### 条款 23：宁以 non-member、non-friend 替换 member 函数

对于那些只访问类的成员函数的函数，考虑用 non-member 和 non-friend 函数替换，这会有更好的封装性和弹性。

```cpp
class WebBrower
{
public:
    void clearCache();
    void clearBookMark();
    void clearUrls();

    void clearEverything() { // bad: 作为成员函数，能够访问成员，降低了封装性
        clearBookMark();
        clearCache();
        clearUrls();
    }
};

// good：作为 non-member，只执行成员函数而不能访问成员，封装性更好
// 并且该函数还可以作为其他类的成员函数，例如 Utils，只要不增加成员访问
void clearEverything(WebBrower &wb) {
    wb.clearBookMark();
    wb.clearCache();
    wb.clearUrls();
};
```

而对于上述的 non-member 的管理，比较自然的做法是放到同一个 namespace 中

```cpp
namespace WebBrowerStuff {
class WebBrower{...}

void clearEverything(WebBrower &wb);
}
```

随着类似函数的增加，我们可以将其根据相关性分割到不同的头文件当中管理，需要哪个的话只需要导入对应头文件而不需要全部导入。这也是标准库的组织方式。
​

这样的形式很方便我们在此基础上拓展新的模块，并且降低了彼此之间的编译依赖性。

```cpp
// webborwer.h
namespace WebBrowerStuff {
class WebBrower{...}
// 一些通用的工具函数
}

// webcache.h
namespace WebBrowerStuff {
}

// webbookmark.h
namespace WebBrowerStuff {
}

// main.cpp
#include "bookmark.h"
int main()
{
    WebBrowerStuff::bookMarkUtil();
}
```

### 条款 24：若所有参数皆需要类型转换，请为此采用 non-member 函数

令 class 支持隐式转换通常是不好的，但数值类型是例外。例如一个有理数的类：

```cpp
class Rational {
public:
    Rational(int numberator = 0, int denominator = 1); // 不为 explicit ，支持隐式转换
    const Rational operator* (const Rational & rhs) const;
};
```

上述的 Rational 相乘没有问题，但是当你要支持混合计算就会出问题：

```cpp
Rational one(1,2);
Rational result;
result = one * 2; // 函数形式：one.operator*(2), 2 发生隐式转换
result = 2 * one; // 函数形式：2.operator*(one), 不发生类型转换
```

对此的解决方式是，通过 non-member 函数来让参数进行隐式转换

```cpp
const Rational operator* (const Rational & lhs, const Rational & rhs);
```

### 条款 25：考虑写出一个不抛异常的 swap 函数（C++11）

在 **C++11 中新增了移动语义**（P470），因此就没有原作中写这个条款的理由，出于学习目的写下记录。
​

- 但 std::swap 对你效率不高时，提供一个 swap 成员函数，并保证不抛出异常
- 提供成员函数 swap，应当也提供非成员版本，和对 std::swap 的特化版
- 调用 swap 针对 std::swap 使用 using，然后不带 std 的调用
- 为用户定义类型定义 std template 全特化是好的，但不要加入某些对 std 全新的东西
