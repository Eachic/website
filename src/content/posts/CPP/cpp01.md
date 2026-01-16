---
title: 每日c++ ：const 的几种用法
published: 2026-01-15
tags: [Learning,C++]
category: C++
draft: false
---

# 每日c++: const与static

## const 的几种用法

### 修饰普通变量
被const修饰的变量只能读，不能改变,例：
```c++
const int N = 1e5+10;
```
注意如果全局变量加上const后，作用域会变成文件作用域。也就是说如果想在其他文件访问该全局变量必须使用extern关键词
### 修饰指针

修饰指针有两种修饰的方法，一种为叫常量指针，一种叫指针常量。  
常量指针是指指针指向的对象不能修改，例：
```c++
const Person* p = new Person();
// 此后person对象不能修改
```
指针常量是指这个指针永远指向该对象，不能再被赋值。
```c++
Person* const p = new Person()
```
两个修饰的位置不同
### 修饰函数参数

修饰函数参数可以防止函数内部不小心修改了传入参数
使用方法如下：
```c++
void printToScreen(const Message &msg);
```
在该函数内部，不允许对msg进行修改

### 修饰函数返回值
如果函数返回一个引用或者一个指针，若无const修饰，那么该返回值可以被修改，也就是一个左值，能够被赋值。如果加上const 就无法被赋值，为一个右值。
```c++
int& getX(){
    return x;
}

getX() = 1; // 能够赋值


const int& getY(){
    return y;
}

getY() = 1 ; //不能赋值
```
### 修饰函数本身
一般用在类里面，表示该成员函数没有对成员变量进行修改；
```c++
int getX() const {return x;}
```



