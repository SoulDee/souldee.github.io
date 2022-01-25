---
title: Part 2：构造/析构/赋值运算
urlname: gu4y9y
date: '2022-01-24 12:00:02 +0800'
categories:
  - 编程
  - C++
  - Effective C++
tags:
  - C++
  - 技巧
---

​

构造、析构和赋值运算是一个对象的开始，释放和拷贝，我们应当了解其中发生什么，编译器又会对它们做什么处理，更重要的是学会哪些行为不应该发生在这些函数中的，从而避免一些错误的产生。
​

<!-- more -->

### 条款 05：了解 C++ 默默编写并调用哪些函数

如果你设计了一个类，但并没有声明其构造函数、拷贝构造函数、析构函数和拷贝赋值操作符。那么编译器会为你生成对应函数的默认版本，该版本是 public 且 inline 的。
​

但是只适用于简单的类，当类变得复杂就容易出现问题。

**对于构造函数存在三个问题：**
​

其一：编译器只在没有其他构造函数时才生成默认构造函数，如果你仍然需要默认构造函数就必须使用 default 修饰来告诉编译器：我们有构造函数，但是仍然需要默认构造函数。（P237）

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

尽管编译器会为我们合成默认版本，但是某些类不能依赖这些版本，比如管理动态内存的类，合成的拷贝构造函数会简单的赋值指针，然后析构进行 delete ，如果存在多个对象，这将会导致多次的 delete 同一个指针。（P447）

**在 C++11 还增加了能够大幅度提升性能的移动构造函数和移动赋值运算符：（P473）**

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

### 条款 06：若不想使用编译器自动生成的函数，就该明确拒绝（C++11）

正如条款 05 所说，编译器会为我们生成默认版本的函数，但有时我们明确知道不需要这些函数， 例如 Singleton 模式或者更具体的 iostream，我们不希望对象被拷贝，因此应当明确的拒绝。在 **C++ 11 中可以使用 delete 修饰**来实现。（P449）
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

### 条款 07：为多态基类声明 virtual 析构函数（C++11）

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

但也不应当为所有类实现虚析构函数，实现虚析构函数会导致对象体积的增加。是否需要的判断标准是：**这个类是否至少有一个虚函数，没有的话表明该类并不希望作为基类被继承。**
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

抛出异常会将程序控制权交给异常处理函数，而析构函数的责任就在于释放资源，倘若抛出异常就会有资源没有释放，将造成资源泄漏。

```cpp
MyClass::~MyClass() {
    try {
        throw std::runtime_error("error");
        std::cout << "after throw" << std::endl; // 没有执行
    }  catch (const std::runtime_error &e) {
        std::cout << e.what() <<std::endl;
    }
}
```

​

即便你将异常抛出放在最后，还是会出现问题，原因在于抛出异常后会调用已创建局部对象的析构函数。假定一个 vector 被销毁，第一个 widget 析构异常，而后续的 widget 又抛出异常，C++ 异常机制不能处理同时存在的异常，这会导致结束程序或者不明确行为。
​

因此要阻止异常离开析构函数，解决方案如下：
​

1. 调用 std::abort 来终止程序，抢先终止不确定行为
1. 吞下异常，只做异常的记录，但是这相当于隐瞒了“失败”
1. 重新设计类，将需要在析构终止的内容通过一个接口暴露给用户手动调用，这样的方式至少在异常发生时给了使用者一个对异常作出处理的机会。

### 条款 09：绝不在构造和析构过程中调用 virtual 函数

​

初始化一个派生类会先执行基类构造，然后再派生类构造。这意味着期间调用 virtual 函数，将调用基类版本的 virtual 函数，而不是我们实际想要的是派生类 virtual 函数。
​

原因在于，在基类构造期间派生类的成员未初始化，调用派生类的 virtual 函数将会导致未定义行为，更根本的解释这是，对象的类型在基类构造期间是基类，直到派生类构造开始才被 virtual 和 dynamic_cast 看做派生类。对于析构函数相同，先析构派生类，然后析构基类。

```cpp
class Base {
public:
    Base() { fn1(); }
    ~Base(){ fn2(); }
    virtual void fn1() { std::cout << "Constructor for class base" << std::endl; }
    virtual void fn2() { std::cout << "Deconstruction for class base" << std::endl;}
};

class Derived: public Base {
public:
    Derived(){ fn1();}
    ~Derived(){fn2();}
    void fn1() override {std::cout << "Constructor for class Derived" << std::endl;}
    void fn2() override {std::cout << "Deconstruction for class Derived" << std::endl;}
};


// result:
// Constructor for class base
// Constructor for class Derived
// Deconstruction for class Derived
// Deconstruction for class base
```

通常来说编译器会对我们发出警告，但是有时候我们隐晦的在构造和析构调用 virtual 函数将不一定会被编译器发现，例如使用构造调用函数 init，然后在 init 中调用 virtual 函数。

```cpp
class Base {
public:
    Base() { init(); }
    void init() {
        fn1();
    }
	// ...
};
```

​

### 条款 10：令 operator= 返回一个 reference to \*this

令赋值相关运算符返回 reference to \*this 能够让我们实现链式赋值，这在内置类型和标准库都得到了广泛使用
​

```cpp
class MyClass {
public:
    MyClass &operator=(const MyClass &rhs) {
        // do something
        return *this;
    };
};

int main()
{
    MyClass a, b, c;
    a = b = c;
}
```

​

​

### 条款 11：在 operator= 中处理“自我赋值”

​

自我赋值就是将自己赋值给自己

```cpp
int a = 1;
a = a;
```

编译器会发出警告，但是这是合法的。通常我们不会这么做，但是在 C++ 中对象是可以拥有“别名”的，因此有时候“不小心”作出这样的行为：

```cpp
// 实际指向同一个对象
array[i] = array[j];
*p1 = *p2;
```

​

倘若自行管理资源，就可能导致“在停止使用资源前意外释放了它”：

```cpp
class Bitmap {};
class Widget {
public:
    // 赋值语句：widget2 = widget1，此时如果 widget1 和 widget2 为同一对象
    // （1）将释放 p，然后（2）又马上使用了刚释放的 p
    Widget &operator=(const Widget &rhs) {
        delete p;                // （1）widget2 停止使用 p
        p = new Bitmap(*rhs.p);  // （2）以 widget1 的 p 来初始化 widget2 的 p
        return *this;
    }

private:
    Bitmap* p;
};
```

阻止这种错误的传统做法是进行“证同测试”，但这不具有“异常安全性”，应当先复制再进行删除

```cpp
// 证同测试
class Widget {
public:
    Widget &operator=(const Widget &rhs) {
        if(this == &rhs) return *this;

        delete p; // (1) 删除 p
        p = new Bitmap(*rhs.p); // 若 new Bitmap 发生异常，这 p 指向（1）已经删除的指针，此类指针是有害的
		return *this;
    }
};

// 异常安全版本
class Widget {
public:
    Widget &operator=(const Widget &rhs) {
        Bitmap* pb = p;	// 保存原 bitmap 指针副本
        p = new Bitmap(*rhs.p); // 若发生异常，p 还是指向原来的 p
        delete pb; // 无异常发生，则通过副本释放原来 bitmap 对象
        return *this;
    }
};
```

###

### 条款 12：复制对象时勿其每一个成分

当我们声明了自己的拷贝构造函数和拷贝赋值运算符，编译器将不为我们生成默认版本，也不会检查为我们声明的版本检查是否遗漏了成分。因此当我们自行声明了拷贝函数，之后新添加成员，都应当记得在拷贝函数添加成员拷贝的步骤。
​

在继承体系中，派生类不能访问基类的私有成员，因此要调用基类的拷贝函数来拷贝基类成分。

```cpp
class Base {
public:
    Base(const Base &rhs): name(rhs.name) {}
    Base &operator=(const Base &rhs) {
        name = rhs.name;
        return *this;
    }

private:
    std::string name;
};

class Driverd: public Base {
    Driverd(const Driverd &rhs)
        :Base(rhs), // 拷贝基类成分
    	 age(rhs.age) {};
    Driverd &operator=(const Driverd &rhs) {
        Base::operator=(rhs); // 拷贝基类成分
        age = rhs.age;
        return *this;
    }
private:
    int age;
};
```

​

此外，不要为了消除重复代码在拷贝赋值运算符调用拷贝构造函数，这相当于试图构造已经存在的对象，反过来调用也不行，相当于对尚未构造好的对象进行赋值。正确的做法是将重复代码提取成一个 private init 的成员函数。
