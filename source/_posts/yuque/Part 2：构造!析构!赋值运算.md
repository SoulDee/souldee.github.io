---
title: Part 2：构造/析构/赋值运算
urlname: gu4y9y
date: '2022-01-10 22:50:13 +0800'
categories:
  - 编程
  - C++
  - Effective C++
tags:
  - C++
  - 技巧
---

​

构造/析构/赋值运算
​

<!-- more -->

### 条款 05：了解 C++ 默默编写并调用哪些函数

如果你设计了一个类，但并没有声明其构造函数、拷贝构造函数、析构函数和拷贝赋值操作符。那么编译器会为你生成对应函数的默认版本，该版本是 public 且 inline 的。
​

但是只适用于简单的类，当类变得复杂就容易出现问题。

**对于构造函数存在三个问题：**
​

其一：编译器只在没有其他构造函数时才生成默认构造函数，如果你仍然需要默认构造函数就必须使用 default 修饰来告诉编译器：我们有构造函数，但是仍然需要默认构造函数。

```cpp
class MyClass {
    MyClass() = default;
}
```

其二：位于块内的内置类型或复合类型（数组、指针）对象的默认初始化后值是未定义的，这在默认的构造函数中同样适用。

```cpp
int a; // 被初始化为 0

int main()
{
    int b; // 未定义
}
```

其三：如果类的成员没有默认构造函数，那么编译器无法初始化该成员。
​

**对于拷贝构造函数，析构函数和拷贝赋值运输符：**
​

尽管编译器会为我们合成默认版本，但是某些类不能依赖这些版本，比如管理动态内存的类，合成的拷贝构造函数会简单的赋值指针，然后析构进行 delete ，如果存在多个对象，这将会导致多次的 delete 同一个指针。

**在 C++11 还增加了能够大幅度提升性能的移动构造函数和移动赋值运算符：**

```cpp
class MyClass {
public:
    // 移动操作不应该抛出异常
    MyClass(MyClass &&c) noexcept;
    MyClass &operator=(MyClass &&c) noexcept;
};
```

编译器同样会为他们生成默认的版本。但是有所区别的是，当类定义了拷贝构造函数、拷贝赋值运算符或者析构函数，编译器不会为其合成默认版本，只有当一个类没有定义自己的考本控制，且每个非 static 成员都可移动，编译器才会自动生成默认版本。
​

​

### 条款 06：若不想使用编译器自动生成的函数，就该明确拒绝

正如条款 05 所说，编译器会为我们生成默认版本的函数，但有时我们明确知道不需要这些函数， 例如 Singleton 模式或者更具体的 iostream，我们不希望对象被拷贝，因此应当明确的拒绝。在 C++ 11 中可以使用 delete 修饰来实现。
​

```cpp
class MyClass {
public:
    MyClass(const MyClass&) = delete;
    MyClass& operator=(const MyClass&) = delete;
};
```

在 C++ 11 之前，则是通过 private 且仅声明为不定义来阻止拷贝：

```cpp
class MyClass {
private:
    MyClass(const MyClass&);
    MyClass& operator=(const MyClass&);
};
```

​

​

### 条款 07：为多态基类声明 virtual 析构函数

当一个派生类通过基类指针被删除，且基类的析构函数为 non-virtual，那么结果未定义，派生类的成分可能不被销毁。

```cpp
class Base {
public:
    ~Base();
};
class Derived: public Base {...};

Base *getClass() {
    return new Derived;
};

int main()
{
    auto p = getClass();
    delete p; // 只是销毁了 Base 成分，而 Derived 成分为销毁。
}
```

解决方式是对基类析构使用 virtual 修饰，这将使这个函数能被覆盖且动态绑定，在运行时确定使用哪个函数。

```cpp
class Base {
public:
    virtual ~Base();
};
```

但也不应当为所有类实现虚析构函数，实现虚析构函数会导致对象提及的增加。是否需要的判断标准是：**这个类是否至少有一个虚函数，没有的话表明该类并不希望作为基类被继承。**
​

对于一些类来说，例如 STL 的容器类，尽管自身没有虚析构函数，但如果使用者错误的继承这些类，同样会导致上述问题，解决的办法是使用 C++ 11 的 **final** 说明符来阻止继承。

```cpp
class Base final {}
```

​

此外，对于一个派生类覆盖的虚函数，应当明确的使用 override 说明符，否则很难调试，例如派生类中同名函数，但是参数列表顺序不同，编译器可能就警告但并不会报错。

```cpp
class Base {
protected:
    virtual void doSomething(int a, bool b);
};
class Derived: public Base {
    void doSomething(bool b, int a); // 没有下方 override 版本仍然可以通过编译
    void doSomething(int a, bool b) override;
};
```

​

### 条款 08：别让异常逃离析构函数

C++ 的异常机制不能同时处理两个及以上的异常，多个异常同时如果程序不结束将会导致不确定的行为。
​

而如果在析构函数中将异常抛出将导致异常传播，那么就会导致上述的多个异常同时存在，解决方式有三种：
​

1. 调用 std::abort 来终止程序，抢先终止不确定行为
1. 吞下异常，只做异常的记录，但是这相当于隐瞒了“失败”
1. 重新设计类，将需要在析构终止的内容通过一个接口暴露给用户手动调用，这样的方式至少在异常发生时给了使用者一个对异常作出处理的机会。

###

对使用 C++ 异常处理应具有怎样的态度？ - 陈硕的回答 - 知乎 https://www.zhihu.com/question/22889420/answer/22975569

### 条款 09：绝不在构造和析构过程中调用 virtual 函数

​

​

​

### 相关链接

- [《C++ Primer》](https://book.douban.com/subject/25708312/)
- [《Effective C++》](https://book.douban.com/subject/1842426/)
- [重述《Effective C++》](https://normaluhr.github.io/2020/12/31/Effective-C++/)
- [对使用 C++ 异常处理应具有怎样的态度？- 知乎](https://www.zhihu.com/question/22889420)
- [Exceptions and Error Handling - isocpp](https://isocpp.org/wiki/faq/exceptions)
- [Error and Exception Handling - boost](https://www.boost.org/community/error_handling.html)

​

​

​
