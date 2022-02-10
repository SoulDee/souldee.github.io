---
title: Part 0：目录
urlname: mbnz1e
date: '2022-02-09 23:24:14 +0800'
categories:
  - Program
  - Cpp
  - More Effective Cpp
tags:
  - C++
  - 技巧
---

本系列为学习《More Effective C++》这本书的笔记，个别内容会增加 C++11 的部分。主要从《C++ Primer 第 5 版》得来。为了加深个人的理解一般自己编写一个类似的示例，但有一些则是直接从书中摘抄。
​

PS：文中出现 P123 这样的标识通常是《C++ Primer 第 5 版》相关内容的页数。
PS：文中条款后写有 （C++11）的意味着在 C++11 版本中已经有了更好的解决方式，但是出于学习的目的，还是对之前版本的思路记录笔记。
PS：对于理解的地方通常会写代码示例，但一些不理解的地方还是会先写下文本记录，以供日后重新思考。
​

个人水平有限，如有错误，欢迎指正帮助我进步。

<!-- more -->

点击这里查看笔记：[More Effective C++](/categories/Program/Cpp/More-Effective-Cpp/)

## Table of contents

- Part 1：基础议题（施工中...）
  - 条款 01：仔细区别 pointers 和 references
  - 条款 02: 最好使用 C++ 转型操作符
  - 条款 03: 绝对不要以多态（polymorphically）方式处理数组
  - 条款 04: 非必要不提供 default constructor
- Part 2：操作符（未开始）
  - 条款 05：对定制的“类型转换函数”保持警觉
  - 条款 06：区别 increment/decrement 操作符的前置和后置
  - 条款 07：千万不要重载 &&，|| 和 , 操作符
  - 条款 08：了解各种不同意义的 new 和 delete
- Part 3：异常（未开始）
  - 条款 09：利用 destruction 避免泄漏资源
  - 条款 10：在 construction 内阻止资源泄漏（resource leak）
  - 条款 11：禁止异常（exceptions）流出 destruction 之外
  - 条款 12：了解“抛出一个 exception”与“传递一个参数”或“调用一个虚函数”之间的差异
  - 条款 13：以 by reference 方式捕捉 exceptions
  - 条款 14：明智运用 exception specifications
  - 条款 15：了解异常处理（exception handling）的成本
- Part 4：效率（未开始）
  - 条款 16：谨记 80-20 法则
  - 条款 17：考虑使用 lazy evaluation（缓式评估）
  - 条款 18：分期摊还预期的计算成本
  - 条款 19：了解临时对象的来源
  - 条款 20：协助完成“返回值优化（RVO）”
  - 条款 21：利用重载技术（overload）避免隐式类型转换（implict type conversions）
  - 条款 22：考虑以操作符复合形式（op=）取代其独身形式（op）
  - 条款 23：考虑使用其他程序库
  - 条款 24：了解 virtual functions、multiple inheritance、virtual base classes、runtime type identification 的成本
- Part 5：技术（未开始）
  - 条款 25：将 constrctior 和 non-member function 虚化
  - 条款 26：限制某个 class 所能产生的对象数量
  - 条款 27：要求（或禁止）对象产生与 heap 之中
  - 条款 28：Smart Pointers（智能指针）
  - 条款 29：Reference counting（引用计数）
  - 条款 30：Proxy classes（替身类、代理类）
  - 条款 31：让函数根据一个以上的对象类型来决定如何虚化
- Part 6：杂项谈论（未开始）
  - 条款 32：在未来时态下发展程序
  - 条款 33：将非尾端类（non-leaf classes）设计为抽象类（abstract classes）
  - 条款 34：如何在同一个程序中结合 C++ 和 C
  - 条款 35：让自己习惯于标准 C++ 语言
