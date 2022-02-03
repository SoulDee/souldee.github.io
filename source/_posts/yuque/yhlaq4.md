---
title: Part 5：实现
urlname: yhlaq4
date: '2022-01-24 12:00:05 +0800'
categories:
  - 编程
  - C++
  - Effective C++
tags:
  - C++
  - 技巧
---

描述了代码实现的一些细节，变量应该什么时候定义，转型的问题，为异常安全付出能够得到的回报，如何使用 inline 提高效率但又不能过度滥用，以及如何进行解耦。但是条款 30 和 条款 31 有点看不明白，因为对于编译的整个流程并不清楚。

<!-- more -->

### 条款 26：尽可能延后变量定义式的出现时间

用到之前才定义变量能够减少不必要的构造函数成本

```cpp
void func(bool isChecked) {
    MyClass c; // bad：如果下方 isChecked 为 true 则 MyClass 的构造多余

    if(!isChecked)
        return;

    c.doSomething();
}

void func(bool isChecked) {

    if(!isChecked)
        return;

    MyClass c; // good：不用到就没有构造
    c.doSomething();
}
```

对于需要使用参数赋值初始化的，应当使用参数作为初值从而避免一个默认构造函数的调用

```cpp
// 假设 MyClass 为一个可以使用 isChecked 构造的类
void func(bool isChecked) {
    ...
    MyClass c; // bad：默认构造 + 赋值构造
    c = isChecked;
    ...

    MyClass c(isChecked); // good：一个构造函数调用
}
```

而对于循环，变量可以定义在循环体外也可以定义在循环体内，需要根据赋值成本高低来选择，但会影响一定的可理解性和维护性，因此除非当你知道赋值成本低于构造加析构或者处理要求高性能的部分，否则应当定义在循环体内。

```cpp
int main()
{
    // 成本：1 个构造 + 1个析构 + n 个赋值
    MyClass c;
    for(int i = 0; i < n; ++i) {
        c = // 某个值
    }

    // 成本：n 个构造 + n 个析构
    for(int i = 0; i < n; ++i) {
        MyClass c(某个值);
    }
}

// 假设成本构造为 x，析构为 y, 赋值为 z，循环次数为 n
// x + y + zn > xn + yn
// z > x + y - (x + y)/n
// 随着 n 的增大 (x+y)/n 部分影响接近0
// 因此 z 也就是赋值成本大于一组构造+析构时在循环体内更高效
```

### 条款 27：尽量少做转型动作

在 C++ 中有 C 风格转型和 4 种新式转型，新式转型更容易识别和排查问题，并且都各自具有专门的职能，应当尽量使用新式转型。

- C 风格：(T)expression 或者 T(expresstion)，在 C++ 中兼容可用
- const_case：常用于常量性移除，即 const 转为 non-const
- dynamic_case：“安全的向下转型”，基类转派生类，成本较大
- reinterpret_case：低级转型，例如将 pointer to int 转为 int，实际动作取决于编译器，因此不可移植。
- static_case：隐式转换，但不能去除常量性

​

类型转换是真的会令编译器编译出运行期间的代码，而根据对象的布局方式不同会有不同行为。编写代码时不应当以“知道对象布局”为基础转型，这会导致其他平台不一定可行。

```cpp
// int 和 double 底层表述不同
int x, y;
double d = static_case<double>(x)/y

// 有时候 Derived* 会被施加一个偏移量来取得 Base*，意味着单一对象可能有两个地址
Derived d;
Base *pb = &d;
```

如果需要在派生类调用基类的函数，正确做法是调用基类函数而不是将派生类转型后调用，因为这种做法实际上调用的是“\*this 对象之 base class 成分”的暂时副本。

```cpp
class Base {
public:
    virtual void add() {
        counter++;
    };
    int getCounter() const {
        return counter;
    };
private:
    int counter = 0;
};

class Derived: public Base {
public:
    void add() override {
        Base::add(); // good：counter 为 1
        static_cast<Base>(*this).add(); // bad：counter 没有变化，为 0

        // do something for Derived
    };
};
```

dynamic_case 的许多实现版本执行速度很慢，例如基于“class 名称之字符串比较”，对于多层深度的继承体系，将会多次调用 strcmp。
通常情况下使用 dynamic_case 是我们认为需要执行派生类对象的成员函数，但手里只有基类指针或引用，对此有两个做法来避免 dynamic_case。

1. 使用容器，在其中存储指向派生类的指针，但同一容器只能单种派生类指针，对于多种派生类指针需要多个容器。
1. 在基类定义一个什么也没做的缺省函数供派生类调用，但缺省代码是个馊主意（条款 34）

​

上述两种方式是可替代方案，但做不到完全解决，不过对于 dynamic_case 有一个必须避免的，就是 “连串的 dynamic_case”

### 条款 28：避免返回 handles 指向对象内部成分

handles 指迭代器、指针或引用，通过返回的内部成员或者成员函数 handles，外部能够通过这些 handles 对内部成分进行修改，这将会降低封装性，所以对于返回成员成分的应当明确写上 const。

```cpp
class Rectangle {
public:
    // bad：返回引用外部可修改
    Point& upperLeft() const {return pData->ulhc; };

    // good：const 阻止修改
    const Point& lowerRight() const {return pData->lrhc; };

private:
    std::shared_ptr<RectData> pData;
};
```

即便声明了 const ，但还是可能带来空悬指针

```cpp
const Rectangle boundingBox(const GUIObject &obj);

int main()
{
    GUIObject *obj;

    // 1. boundingBox 返回一个临时的 Rectangle
    // 2. upperLeft 返回这个临时 Rectangle 的内部 const Point&
    // 3. 取地址获得 Point 的指针
    // 4. 语句结束，Rectangle 被析构，包括 Point
    // 5. 由于已经 Point 已经被析构，返回给 pUpperLeft 实际是一个空悬指针
    const Point* pUpperLeft = &(boundingBox(*obj).upperLeft());
}
```

### 条款 29：为”异常安全“而努力是值得的

“异常安全”的函数符合下面两点：
**不泄漏任何资源：**发生异常后是否有资源没有得到释放，可以参考**条款 13** 的示例和防范方式。

```cpp
// c 没有释放，资源泄漏
void func() {
    auto c = new MyClass;
    // 这里发生异常
    ...
}
```

**不允许数据败坏：**函数在执行了一些操作后执行了数据的变更，然后发生了异常，此时操作是失效而数据却发生了改变。

```cpp
// 发生了异常，函数失败，但 counter 还是计数了
void add() {
    ++counter;
    // 这里发生异常
    ...
}
```

​

“异常安全”函数应当提供以下保证之一：

- 基本保证：异常抛出后没有对象或者数据败坏，任何事物应当处于有效状态。
- 强烈保证：异常抛出，状态不改变，即如果调用成功状态改变否则回退到之前的状态。
- 不抛掷保证：函数不抛出异常，总能完成承诺的功能。

要实现强烈保证的”全有或全无“可以采用 copy-and-swap 的策略，即拷贝一个副本，然后在副本上进行修改，修改成功后在进行不抛出异常的 swap

```cpp
void PrettyMenu::changeBackground(std::istream& imgSrc) {
    using std::swap;
    Lock ml(&mutex);
    std::shared_ptr<PMImpl> pNew(new PMImpl(*pImpl)); // 获得副本
    pNew->bgImage.reset(new Image(imgSrc)); // 修改副本，就算出现异常也不影响原来数据
    ++pNew->imageChanges;
    swap(pImpl, pNew); // 成功后 swap 数据
}
```

但倘若该函数中还调用了其他的函数，则不一定能获得强烈保证，一个系统的保证等级取决于保证等级最低的那个函数。

### 条款 30：透彻了解 inlining 的里里外外

- inline 函数会减少函数调用，编译器会对 inline 函数进行语境相关优化。
- inline 背后的理念是“对此函数的每一个调用”都以函数本体替换，因此也会带来代码大小增加。
- inline 是对编译器的申请而不是强制的
- 定义于 class 定义式内的函数是隐喻的 inline 函数
- 将大多数 inline 函数限制在小型、被频繁调用的函数身上
- 不要因为 function templates 出现在头文件，就将它们声明为 inline。

​

### 条款 31：将文件间的编译依存关系降至最低

文件的编译依赖关系会导致当一个头文件改变，其他所有依赖它的文件都要重新编译，这意味着有时候仅仅是一个小修改也要花费很长时间编译。

```cpp
class Person {
    ...
private:
    // 实现细节，当 Data 头文件修改，则包含了 Person 的该文件也要重新编译
    Data data;
};

```

前置声明可以避免导入细节，但是对于标准程序库前置声明太复杂，并且通常标准程序库不会导致你的重新编译，而想要声明自定义类又需要知道它的实现细节。
​

我们可以通过 pimpl idiom 的来解决这个问题，将一个类进行拆分，一个负责提供接口，一个负责实现该接口。

```cpp
// 1. 可以只靠声明定义出指向该类型的指针或引用，但如果是对象则需要定义式
// 2. 当你声明一个函数只需要声明式而无需定义式
// 3. 为声明式和定义式提供不同头文件

// person.h
#include <memory>

class PersonImpl; // 前置声明
class Person
{
public:
    //...
    int age();
    // ...各种接口
private:
    std::shared_ptr<PersonImpl> pImpl; // 仅保存 impl 类的指针
};

// person_p.h
#include <string>

class PersonImpl {
public:
    PersonImpl(const std::string &name, int age);
private:
    std::string name;
    int age;
    // ...
};
```

除此之外还有一种 Interface class 也能实现声明和定义的分离。
​
