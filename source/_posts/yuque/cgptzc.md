---
title: Part 6：继承与面向对象设计
urlname: cgptzc
date: '2022-01-24 12:00:06 +0800'
categories:
  - 编程
  - C++
  - Effective C++
tags:
  - C++
  - 技巧
---

讲述了一些继承体系设计的要点，public 和 private 继承各自意味着什么？在继承中容易导致意料之外行为的行为，基类和派生类之间可能的关系，除了 virtual 函数之外还有什么选择？多重继承可能的歧义和用途等内容。

<!-- more -->

### 条款 32：确定你的 public 继承塑膜出 is-a 关系

public 继承意味着 is-a 关系，即派生类同时是基类对象，要全盘接受基类的所有可继承内容，而反过来则不成立。除此之外还有 has-a**(条款 38)**和 is-implemented-in-terms-of**(条款 39) **。
​

**世界上并不存在适用于所有软件的设计**，在一个软件设计中企鹅是否会飞可能是重要的，因此你需要设计出“飞行鸟类”作为区分来使得不能飞的企鹅合理，然而在另一个软件设计中这可能并不重要，在这个软件系统中或许更关心企鹅的翅膀等特征。
​

在 is-a 继承体系时不要被旧有知识和直觉所囚禁，例如在我们的认知中，正方形是特殊的矩形，应当继承自矩形，然而在 is-a 中这确不对，正方形的宽高不能独立的修改，而矩形却可以。

### 条款 33：避免遮掩继承来的名称

派生类内的名称遮掩基类名称，包括那些名称相同但参数不同的函数，而根据**条款 32 **我们知道 public 继承应当是 is-a 关系，基类被继承函数应该得到都要体现。

```cpp
class Base {
public:
    virtual void func();
    virtual void func(int n);
};

class Derived: public Base {
public:
    virtual void func();
};

int main()
{
    Derived d;
    int n = 10;
    d.func(n); // error：Base::func(int n) 也被遮掩找不到了
}

```

如果想要让被遮掩的函数重新可见，可以使用 using

```cpp
class Derived: public Base {
public:
    using Base::func; // 让基类该名称全部可见
    virtual void func();
};
```

有时你并不想继承所有函数，那么不应该使用 public 继承，而是使用 private 继承，同时使用一个转交函数

```cpp
class Derived: private Base {
public:
    virtual void func(){ Base::func(); }; // 转交并使其成为 inline 函数
};
```

### 条款 34：区分接口继承和实现继承

继承的时候要区分只继承接口、继承接口和实现和继承接口覆写实现。

```cpp
class Shape {
public:
    // 纯虚函数，意味着这是抽象类，派生类只继承接口，需要自己定义实现
    // 但也可以提供一份实现来作为派生类的缺省行为
    // 可以通过 derivedObj->Shape::draw() 这样的形式调用
    virtual void draw() = 0;

    // virtual，可被继承并覆写，或者作为缺省行为
    virtual void error(const std::string &msg);

    // non-virtual，不应该被特化的，实现方式要统一的函数
    int objectID();
};
```

对于 virtual 函数提供的缺省行为也要小心，可能造成期望之外的行为。

```cpp
class Airport;
class Airplane {
public:
    virtual void fly(const Airplane &destination) {};
};

class ModelA: public Airplane {
    // 有时候可能是我们忘了自己实现，而此时就会调用缺省，但不一定是我们期望的行为
};
```

对此我们可以重新设计来让编译器帮我们避免这种问题，将缺省行为作为非虚函数提供，当需要缺省行为则主动调用。

```cpp
class Airport;
class Airplane {
public:
    // 纯虚函数，派生类必须有自己实现
    virtual void fly(const Airplane &destination) = 0;

protected:
    void defaultFly(const Airplane &destination){};
};

class ModelA: public Airplane {
public:
    virtual void fly(const Airplane &destination){
        defaultFly(destination); // 需要缺省行为时主动调用
    };
};
```

或者我们也可以利用上述所说的，纯虚函数也可以提供实现作为缺省行为，以及纯虚函数必须有定义实现这两点来设计。

```cpp
lass Airport;
class Airplane {
public:
    virtual void fly(const Airplane &destination) = 0;
};

void Airplane::fly(const Airplane &destination) {
    // pure-virtual 的缺省行为
}

class ModelA: public Airplane {
public:
    virtual void fly(const Airplane &destination){
        Airplane::fly(destination); // 主动调用缺省行为
    };
};
```

### 条款 35：考虑 virtual 函数以外的其他选择（C++11）

当你为解决问题寻找设计方法时，还可以考虑 virtual 之外的替代方法。
​

**设计一个视频游戏软件，为人物设计继承体系，人物有健康值，提供一个函数计算健康值**
​

**原始 Virtual 函数设计方案：**​

```cpp
class GameCharacter {
public:
    virtual int healthValue() const;
};
```

**NVI 手法（Non-Virtual Interface）**：主张虚函数总是 private，通过一个 non-virtual public 成员函数调用 virtual private 函数。优点是能够在 virtual 函数的前后做一些工作。

```cpp
class GameCharacter {
public:
    int healthValue {
        // ... 锁定互斥器、日志、验证约束条件、验证函数先决条件等等
        int retVal = doHealthValue();
        // ... 解除互斥器、再次验证约束条件等等
    }
private:
    virtual int doHealthValue() const {};
};
```

**Strategy 模式 Function Pointers 实现：**计算方法同人物无关，人物接受一个函数指针作为实际计算方式，相比 NVI 手法有更好的弹性，对于继承体系下的不同类型可以在**运行期**通过传入不同的函数指针来指定计算方式。缺点是会弱化 class 的封装性。

```cpp
class GameCharacter; // 前置声明
int defaultHealthCalc(const GameCharacter& gc); // 默认计算方法

class GameCharacter {
public:
    // 函数指针类型
    typedef int (*HealthCalcFunc)(const GameCharacter& gc);

    // 构造函数传入函数指针
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
        :healthFunc(hcf){}

    // 计算方法调用该函数指针
    int healthValue() const {
        return healthFunc(*this);
    }
private:
    HealthCalcFunc healthFunc;
};
```

**Strategy 模式 可调用对象 实现：**我们希望实现方式兼容性更强，例如使用成员函数，返回值也不一定必须固定，而是任何可转换为指定类型的类型。这就要用到标准库的可调用对象(P511)，
​

可调用对象包含了：函数、函数指针、lambda 表达式(P346)、bind 创建对象(P354)以及重载了函数调用运算符的类。

```cpp
class GameCharacter;
// 函数
int defaultHealthCalc(const GameCharacter& gc);
// 函数指针
int (*defaultHealthCalc)(const GameCharacter& gc);
// lambda 表达式
auto defaultHealthCalc = [](const GameCharacter&) -> int {...};
// std::bind 适配器
int test(const GameCharacter& gc);
auto defaultHealthCalc = std::bind(test, std::placeholders::_1);

class GameCharacter {
public:
    typedef std::function<int(const GameCharacter& gc)> HealthCalcFunc;
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
        :healthFunc(hcf){}
    ...
};


// 重载了函数调用，用法稍微不同
struct HealthCalculator {
    int operator()(const GameCharacter& gc) const;
};
class GameCharacter {
public:
    ...
    explicit GameCharacter(HealthCalcFunc hcf = HealthCalculator())
        :healthFunc(hcf){}
    ...
};
```

**古典的 Strategy 模式：**会将函数做成一个分离的继承体系中的 virtual 函数。即角色一个继承体系，健康计算函数是另一个继承体系。优点是容易辨认，并且需要其他健康计算方式只需要增加一个派生类。

```cpp
class GameCharacter; // 前置声明

// 健康计算继承体系
class HealthCalcFunc {
public:
    virtual int calc(const GameCharacter& gc) const;
};

HealthCalcFunc defaultHealthCalc;

// 角色继承体系
class GameCharacter {
public:
    explicit GameCharacter(HealthCalcFunc *phcf = &defaultHealthCalc)
        :pHealthCalc(phcf){}
    int healthValue() const {
        return pHealthCalc->calc(*this);
    }
private:
    HealthCalcFunc* pHealthCalc;
};
```

### 条款 36：绝不重新定义继承而来的 non-virtual 函数

non-virtual 函数是静态绑定，因此当你在派生类覆写之后通过一个转换来的基类指针调用该函数，实际调用的是基类的函数。

```cpp
class Base {
public:
    void log() {
        std::cout<< "Base" << std::endl;
    }
};

class Derived: public Base {
public:
    void log() {
        std::cout<< "Derived" << std::endl;
    }
};

int main()
{
    // 指向同一个对象，调用的确实不同版本
    Derived d;
    Base *b = &d;
    b->log(); // base
    Derived *d2 = &d;
    d2->log(); // derived
}
```

此外，根据条款 32，当你使用派生类去覆写 non-virtual 函数将会破坏 pubilc 继承的 is-a 关系。

### 条款 37：绝不重新定义继承而来的缺省参数值

virtual 函数是是动态绑定，而缺省参数值确实静态绑定，因此会出现在派生类调用了该函数，以为使用了派生类的缺省值，实际上使用了基类的缺省值。

```cpp
class Base {
public:
    enum Type { BaseType, DerivedType };
    virtual void log(Type t = Type::BaseType) = 0;
};

class Derived: public Base {
public:
    virtual void log(Type t = Type::DerivedType) {
        std::cout << t << std::endl;
    }
};

int main()
{
    Base *b = new Derived;
    b->log(); // 0, 也就是 BaseType
}
```

### 条款 38：通过复合塑膜出 has-a 或“根据某物实现出”

is-a 意味着派生类必然是基类，而反之则不是，是一种子集和真子集的关系

```cpp
class Base {};
class Derived: public Base {};
```

通过复合类型可以塑造 has-a，指是类型中含有其他成分

```cpp
class Address{...};
class Phone{...};
class Email{...};

// 含有xxx成分
class Person {
private:
    Address address;
    Phone phone;
    Email emial;
};
```

通过复合类型可以塑造 is-implemented-in-terms-of ，即是通过 A 的方式来实现 B

```cpp
// 以 list 模板来实现 Set 的行为
template<class T>
class Set {
private:
std::list<T> rep;
};
```

### 条款 39：明智而审慎地使用 private 继承

private 继承自基类的所有内容在派生类会成为 private，包括 public 和 protected。
​

private 继承意味着“is-implemented-in-terms-of”，即根据某物实现，是一个实现技术而非设计。
​

通常来说，想要实现“is-implemented-in-terms-of”应当选择**条款 38 **的复合类型，复合类型可以有派生类，并且能够通过前置声明来降低编译依赖。
​

一些特殊情况下才决定使用 private 继承，例如你是致力于“对象尺寸最小化”的程序库开发者，private 能有 empty base 最优化

```cpp
class Empty{};

// sizeof(HoldsAnint) > sizeof(int)
// C++ 默认会安插 char 到空对象
class HoldsAnint {
private:
    int x;
    Empty e;
}

// EMO 空白基类最优化
class HoldsAnint: private Empty {
private:
    int x;
}
```

### 条款 40：明智而审慎地使用多重继承

多重继承中继承相同名称容易导致歧义

```cpp
class BaseA {
public:
    void doSomething();
};

class BaseB {
private:
    void doSomething();
};

class Derived:
        public BaseA,
        public BaseB {
};

int main()
{
    Derived d;
    d.doSomething(); // error: 不知道调用哪个，即便一个 public 一个 private
    d.BaseA::doSomething(); // 可以指定调用，public 成功
    d.BaseB::doSomething(); // private 调用失败
}
```

还有可能导致“钻石型多重继承”，即基类和派生类有一个以上的连通路线，在这里继承中默认会对成员变量都进行复制，例如 File 中有一个 filename 则最终 IOFile 会有两个数据。

```cpp
class File {};
class InputFile: public File {};
class OutputFile: public File {};
class IOFile:
        public InputFile,
        public OutputFile {
};

// 可以使用“virtual 继承”来避免，但这会带来额外成本，体积更大，访问更慢
class InputFile: virtual public File {};
class OutputFile: virtual public File {};
```

多重继承也有其正当的用途，例如“public 继承某个 Interface class” 和 “private 继承某个协助实现的 class” 的组合。（暂略）
