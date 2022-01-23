---
title: Part 0：目录
urlname: cp9lsm
date: '2022-01-23 06:48:21 +0800'
categories:
  - 编程
  - C++
  - Effective C++
tags:
  - C++
  - 技巧
---

本系列为学习《Effective C++》这本书的笔记，原书因为年代比较早，缺少了 C++11 的内容，针对这一点补充一些相关内容，主要从《C++ Primer 第 5 版》得来。
​

PS：文中出现 P123 这样的标识通常是《C++ Primer 第 5 版》相关内容的页数。
​

个人水平有限，如有错误，欢迎指正帮助我进步。
​

<!-- more -->

## Table of contents

- Part 1：让自己习惯 C++
  - 条款 01: 视 C++ 为一个语言联邦
  - 条款 02: 尽量以 const，enum，inline 替换 #define
  - 条款 03: 尽可能使用 const
  - 条款 04: 确定对象被使用前已先被初始化
- Part 2：构造/析构/赋值运算
  - 条款 05：了解 C++ 默默编写并调用哪些函数
  - 条款 06：若不想使用编译器自动生成的函数，就该明确拒绝
  - 条款 07：为多态基类声明 virtual 析构函数
  - 条款 08：别让异常逃离析构函数
  - 条款 09：绝不在构造和析构过程中调用 virtual 函数
  - 条款 10：令 operator= 返回一个 reference to \*this
  - 条款 11：在 operator= 中处理“自我赋值”
  - 条款 12：复制对象时勿其每一个成分
- Part 3：资源管理
  - 条款 13：以对象管理资源
  - 条款 14：在资源管理类中小心 copying 行为
  - 条款 15：在资源管理类中提供对原始资源的访问
  - 条款 16：成对使用 new 和 delete 时要采取相同形式
  - 条款 17：以独立语句将 newed 对象置入智能指针
- Part 4：设计与声明
  - ......

## Refrence

- [《C++ Primer》](https://book.douban.com/subject/25708312/)
- [《Effective C++》](https://book.douban.com/subject/1842426/)
