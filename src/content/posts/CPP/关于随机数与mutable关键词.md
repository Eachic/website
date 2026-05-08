---
title: 关于随机数与mutable关键词
published: 2026-04-19
tags: [C++,软件工程]
category: C++
image: 'https://t.alcy.cc/moez'
draft: false
---

# 关于随机数与mutable关键词

之前一直觉得mutable关键词非常矛盾,为什么要在const关键词中修改成员变量，
如果一定要修改，那么把const 去掉就行了呀。

但是，今天在封装一个随机数类的时候，终于用到了mutable关键词，


## 情景
情景如下：
我需要在我的game 类中封装一个产生随机数的类，同时为了使用c++ ,我打算不使用我比较属熟悉的srand() + rand() 的用法
而是使用c++ 标准库 里面的函数
主要使用方法如下
```c++

#include <random>

std::random_device rd; 
std::mt19937 gen(rd());

std::uniform_real_distribution<float> dist(1,100);
float random_f = dist(gen);
```

然后我的想法是将随机数引擎作为一个成员变量，然后在成员函数中根据分布范围创建分布对象

具体代码如下：
```c++
std::mt19937 gen_ = std::mt19937(std::random_device{}()); // 随机数生成器

float getRandomFloat(float min, float max) const {
    std::uniform_real_distribution<float> dist(min, max);
    return dist(gen_);
}
```

从逻辑上来讲获取一个随机数函数就是不会改变我这个对象的什么状态，所以他理应是const,但是这样无法通过编译，因为，在调用随机数时，随机数引擎是会发生变化的，但是这个变化对我来说我是不关心的，
如果为了这个变化，而去掉getRandomFloat 的const 的属性，我会觉得很奇怪。  
这时候我突然想起了mutable关键词，如果给随机数引擎一个mutable 关键词修饰，这样就能顺利编译通过，同时也不破坏函数逻辑上的const 。

## 总结

我认为从编程哲学而言，mutable 关键词确实是有必要存在的,它可以修饰一些我们不关心其是否改变的成员变量，这个成员变量可是这是别人为了实现某个功能而必须的成员变量，但是我并不关心这个成员变量的实际内部结构。



