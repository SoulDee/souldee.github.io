---
title: Part 3：资源管理
urlname: wq00mc
date: '2022-01-24 00:00:03 +0800'
categories:
  - 编程
  - C++
  - Effective C++
tags:
  - C++
  - 技巧
---

资源来自系统，用完了就要还，否则将会导致各种问题，例如最常见的动态内存管理不当导致资源泄漏问题。常见的资源还有文件描述符、互斥锁、图形界面的笔刷和字体、数据库连接、网络 sockets。
​

<!-- more -->

### 条款 13：以对象管理资源（C++11）

手动的去管理资源很难保证不会出现资源泄漏，一方面在于代码设计上不能保证获取的资源能释放，例如异常导致的控制流等等，另一方面在于未来接管代码的其他人可能并不知道你获取了资源，然后在释放资源之前就进行了 return 等控制流的转交。

```cpp
int main()
{
    MyClass *c = new MyClass;
    // 代码设计原因，或者其他人的新代码导致的控制流转交，delete 未能执行
    delete c;
}

```

因此要将资源交给 RAII 对象进行管理，即该对象在构造中获取资源并马上初始化，并通过析构来释放资源，针对这种情形，在原书中提到是使用 auto_ptr 和 tr1::shared_ptr 。
​

而在 C++11 中 shared_ptr 已经成为标准，还提供了 make_shared 这样的函数。还要注意一点 shared_ptr 的构造函数是 explicit 的，所以不能将内置指针隐式转换为智能指针，必须使用直接初始化而不是拷贝初始化。（P400）

```cpp
auto p = std::make_shared<int>(10); // 直接返回一个 shared_ptr

std::shared_ptr<int> p = new int(10); // Error: 拷贝初始化
std::shared_ptr<int> p(new int(10)); // Success：直接初始化

```

​

auto_ptr 则是使用 unique_ptr 进行替代。（P417-419）auto_ptr 具有唯一性，但是却又可以进行复制的操作，而操作的结果却又会将上一个 auto_ptr 设为 null，这样的行为导致。我们无法在标准容器中保存 auto_ptr(容器要求能正常复制行为)，也不能从函数中返回 auto_ptr。
​

在 unique_ptr 中你不能进行复制和赋值，但可以通过直接初始化或者 release、reset 函数来转交指针。

```cpp
int main()
{
    std::unique_ptr<int> p(new int(10));
    std::unique_ptr<int> p2(p.release()); // p1 转交给 p2
    std::unique_ptr<int> p3(new int(10));
    p3.reset(p2.release()); // p3 释放原资源，p2 转交给 p3
}


// 通过函数返回
std::unique_ptr<int> getUniquePoint() {
    std::unique_ptr<int> p(new int(10));
    return p;
}

int main()
{
    std::unique_ptr<int> p2(getUniquePoint().release());
    std::cout << *p2 << std::endl; // 10
}

```

​

除此之外还有一个 weak_ptr，指向一个 shared_ptr 资源，但是不递增计数器。需要一个 shared_ptr 来进行初始化。weak_ptr 的作用是在不影响资源生存周期的情况下，访问资源，同时阻止用户访问不再存在的资源。（P420）
​

### 条款 14：在资源管理类中小心 copying 行为

​

并非所有资源都是分配到 heap 区的，对于这些资源，如果我们想要建立自己的资源管理类，同样需要遵循 RAII。而在此时需要注意这些类的 copying 行为，有以下的场景和处理方式：
​

**禁止复制：**对于一些类来说，copying 动作并不合理，应当使用 = delete 禁止 copying 函数，例如互斥锁等同步原语。
​

**对底层资源采用“引用计数”：**有时我们希望保有资源，直到无人使用的。这时候就可以使用引用技术的 shared_ptr 了，默认情况下 shared_ptr 默认指向的是动态内存，因此会在析构中 delete 来释放。
​

而对于一些资源来说释放方式并非通过 delete，例如一个数据库的连接，这时候就可以使用 shared_ptr 初始化的第二个参数，传递一个自定义的函数指针来释放资源。（P416）
​

**复制底层资源：**只要你喜欢，你可以对任意资源进行复制，但要记住复制管理资源对象的指针，同时也要复制其内存。
​

此处内存优化可以考虑使用**“写时复制”**技术，即拷贝对象时，只拷贝其指针，直到需要对该内存进行写操作的时候再拷贝一份内存。
​

**转移资源控制权：**使用 unique_ptr 的 Realse/Reset 来进行转移。（P418）
​

​

### 条款 15：在资源管理类中提供对原始资源的访问

有时一些 API 会要求访问资源管理类保存的指针而不是资源管理类。对此 shared_ptr 和 unique_ptr 都提供了 get 方法用于获取，但是要小心使用，若智能指针释放资源，返回的指针指向的对象也会消失。（P401）

```cpp
auto p = std::make_shared<int>(10);
int *i = p.get();
```

而对于我们自己建立的 RAII 资源管理类，同样应该提供 get 函数来访问保存的指针。

```cpp
class Resource {
public:
    // ...
    MyClass *get() const { return c; } // 显示转换函数
private:
    MyClass* c;
};
```

若觉得每次都要调用 get 不够优雅，也可以使用隐式转换函数，但是这也可能带来错误：

```cpp
class FontHandle {
public:
    FontHandle();
};

class Font {
public:
    operator FontHandle() const {return a;};
private:
    FontHandle a;
};

int main()
{
    Font font;
    FontHandle handle = font; // 发生了隐式转换
}
```

​

### 条款 16：成对使用 new 和 delete 时要采取相同形式

但使用 new/delete 来手动管理内存时，要注意指针指向的是一个单一对象还是对象数组，并选择正确的方式来释放资源。
​

```cpp
// 指向单一对象
std::string* str = new std::string("this is string");
delete str;

// 指向对象数组
std::string* strArr = new std::string[100];
delete [] strArr;
```

当你对单一对象调用 delete [] ，或者是对对象数组 delete，结果都是未定义的。
​

### 条款 17：以独立语句将 newed 对象置入智能指针

​

对于以下的（1）处的 handler 函数，编译器的执行顺序是不确定的，可能在执行 new int 的后执行 counter ，如果此时发生异常，指针还没转换为智能指针，这将会导致资源泄漏。因此应当写成 （2）处写法，将资源管理对象的创建独立出来。

```cpp
int counter();
void handler(std::shared_ptr<int> p, int);

int main()
{
    handler(std::shared_ptr<int>(new int(10)), counter()); // （1）


    std::shared_ptr<int> p(new int(10));
    handler(p, counter()); //（2）
}
```
