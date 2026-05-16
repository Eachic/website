---
title: 从零开始学习GO-day03
published: 2026-05-16
tags: [learning]
category: Go开发
image: 'https://t.alcy.cc/moez'
draft: false
---

# 从零开始学习GO day02

今天看了一点Go的输入输出,基本还是和C差不多


## 基本输入

### os

用os库中带的文件描述符来创建输入输出通道。感觉和linux有点类似

主要有三个文件描述符`os.Stdout`,`os.Stdin`,`os.Stderr`
这三个文件描述符都有对应的读写的函数
```go

package main

import (
    "os"
)

func main(){
    os.Stdout.WriteString("hello world")
}

```

### bufio

给底层的io了一个封装，提供了buffer的选项。在写文件和网络编程的时候常用

```go
func write(str string) {
    writer := bufio.NewWriter(os.Stdout)
    defer writer.Flush()
    writer.WriteString(str)
}

```

### fmt

这个用得最多，并且和c用的感觉差不多

主要的几个函数是`Scanf`,`Scanln`,`Printf`,`Println`
并且提供非常强大的格式化工具

这几个函数都返回两个参数,一个是写入或读出的bytes数，一个是error信息


## 关键词
