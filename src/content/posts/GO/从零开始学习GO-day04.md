---
title: 从零开始学习GO-day04
published: 2026-05-18
tags: [learning]
category: Go开发
image: 'https://t.alcy.cc/moez'
draft: false
---

# 从零开始学习GO day04


## 迭代器
今天主要学的是Go语言中的迭代器,感觉实际上非常难理解
和c++ 和 python 中的迭代器有很大区别

在c++ 和 python 中,迭代器基于可迭代对象的。本质还是一种对象,这个对象往往组合了可迭代对象,并且实现了一些特殊方法，在对象内部的变量保存这可迭代对象的迭代状态。  


在go中只要满足 `func( func(T) bool)` 的函数就可以接在 range的后面。 其实感觉自定义迭代器就开发应用而言应该用得还比较少,可能写框架会比较多吧




## 关键词

- range
- 迭代器