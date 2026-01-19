---
title: 每日c++ ：lamdba 表达式
published: 2026-01-19
tags: [Learning,C++]
category: C++
draft: false
---

# 每日c++：lambda 表达式的用法

## lamda 表达式规则：

```c++
[capture](parameters) mutable noexcept -> return_type {
    // 函数体
}
```

其中capture是重点，用于决定Lambda 能否访问外部变量，以何种方式访问
mutable 关键词表示能够修改外部变量
## capture的具体规则

```
[] // 空捕获
[=] // 按值捕获外部所有变量
[&] // 按引用捕获所有外部变量
[x,&y ] // 按值捕获x，按引用捕获y
[this] // 捕获当前类的this指针，方便调用成员函数或变量 
```


