---
title: 每日c++ ：enum 与 enum class的区别
published: 2026-01-17
tags: [Learning,C++]
category: C++
draft: false
---

# 每日c++ : enum 与 enum class的区别

enum class 为c++11的新特性

## 命名空间不同

enum class 中的枚举变量的命名空间默认为枚举类空间，需要枚举类名：：成员名进行访问
enum 中的成员变量能够直接访问

## 转换方式

enum 类别能够隐式转换为int类型，但是 enum class类别不能隐式转换，需要使用 static_cast 方法才能进行转换

## 底层类型

enum 类型的变量默认为int类型，而enum class 类别的变量能够显示指定类型例如:
```c++
enum class Status : char{
    Failure = 'F',
    Success = 'S'
}
```
