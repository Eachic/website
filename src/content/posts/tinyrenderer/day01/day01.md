---
title: tinyrenderer 学习笔记（一）
published: 2025-07-01
tags: [CG,Learning]
category: Tinyrenderer
image: "https://tc.alcy.cc/i/2025/07/29/6887a5642a91b.webp"
draft: false
---
# tinyrenderer 学习笔记（一）

## 简介

开始学习一个简单的光栅化器。从零开始，不借用任何c++标准库外的库,实现一个光栅化器。预计一周时间完成学习

## 初始代码以及环境

初始代码包括三个文件`tgaimage.h`,`tagimage.c`,`main.c` 前面的两个文件主要是用于创建tga格式的图片，tga格式是一种无损位图格式，可以把它视作一个的二维数组。这两个文件给我们了一些接口，不用自己实现。不需要自己实现的原因是重点在于光栅化器的实现，而tga格式的具体规定不是重点。

然后是学习环境。操作系统：Ubuntu20,虚拟机：WSL2，编辑器：vscode。把gcc,make 这些通过apt 安装好后，环境就算搭建好了。

同时为了查看tga 格式的图片，安装imagemagick，使用display 命令查看

## Bresenham`s line drawing

### 基础要求

这一节主要介绍了一种画线的方法。画线看似简单，但其中还是有很多有趣的问题。

我的第一想法，肯定是先把线的解析式求出来，然后for循环x,画线就行了。但是没考虑到斜率的问题。因为像素空间只有整数，如果线的斜率绝对值太大，会导致采样率很低，只有几个点被画出来，考虑极短情况为线与x轴垂直时。

解决方案也很简单，判断一下斜率就行，斜率小于1 则for 循环x 反之，for 循环y。

我的代码如下：

```c++
void _line(int ax,int ay,int bx,int by,TGAImage &framebuffer,TGAColor color,int flag){
    
    
    float  k = (by-ay)*1.0/(bx-ax);
    float  b =  (ay*bx-by*ax)*1.0/(bx-ax);

    for(;ax<=bx;ax++){
        int y = std::round(ax*k + b);
        if(flag){
            framebuffer.set(ax,y,color);
        }else{
            framebuffer.set(y,ax,color);
        }
       
    }
}


void line(int ax,int ay,int bx,int by,TGAImage &framebuffer,TGAColor color){
    
    

    if(std::abs(bx-ax)>std::abs(by-ay)){
        if(bx<ax){
            std::swap(bx,ax);
            std::swap(by,ay);
        }
        _line(ax,ay,bx,by,framebuffer,color,1);
    }else{
        if(by<ay){
            std::swap(bx,ax);
            std::swap(by,ay);
        }
        _line(ay,ax,by,bx,framebuffer,color,0);
    }


}
```

### 性能优化

接着，老师开始谈性能优化的事情。首先`  int y = std::round(ax*k + b);`这一行代码就非常不好，for 循环里面每次都有浮点数乘法，效率很低。这个y可以与x一样，每个for循环加一个数 改成`y += k` 。同时y变为浮点型。

写一个测试程序：

``````c++
  constexpr int width  =512;
  constexpr int height = 512;
    TGAImage framebuffer(width, height, TGAImage::RGB);

    std::srand(std::time(NULL));
    for(int i=0;i<(1<<21);i++){
        int ax = rand()%width, ay = rand()%height;
        int bx = rand()%width, by = rand()%height;
        line(ax, ay, bx, by, framebuffer, { rand()%255, rand()%255, rand()%255, rand()%255 });
    }
    framebuffer.write_tga_file("framebuffer.tga");
    return 0;
``````

使用 linux 系统中的 time 程序计时。优化前：**4.356s ** 优化后：**2.935s**。非常惊人的优化，让我第一次感觉到了乘法有多expensive。

然后老师又谈到了这样会引入浮点数y，每次绘制时，都要进行浮点数到整数的转换，也比较吃性能。然后希望保持整形y，映入一个新的浮点变量error。由于k小于1，而且只有在y累加超过0.5时，y才会进1。因此先把k加到error上，然后每个循环判断error与0.5的大小，以此来判断y是否需要加一。这样就可以保证y的整形不变。具体代码见老师讲义。

然后继续思考，能否把error这个浮点型也优化掉？每个for循环里面 error 会加 一个$k = \frac{by - ay}{bx - ax}$ 同时与 0.5 比大小。那么把error整形化就是整个过程乘以一个$2*(bx-ax)$ 。这样每个for循环就变成了每次加一个$2*(by - ay)$ ,然后与$bx-ax$ 比大小。至此在这个 算法中所有变量均变为了整数。这就是 Bresenham line drawing 算法。

但是由于现代cpu不断进步，浮点运算并没有那么费时了。这样的算法优化浮点运算，带来的结果是更多的分支判断。而恰好现代cpu中分支跳转会破坏流水线，反而使性能下降更为严重。

## 作业

作业是给定一个 obj 格式的几何描述文件，然后画出来。主要就是处理一下obj 格式的文件。

但是c++ 在读文本文件这一块确实库太少了，连分割字符串都要自己写。但是同时也找到一个非常好用的类`std::istringstream` 。相当于cin了。写一个model的类，核心代码如下：

```c++
#include <fstream>
#include <vector>
#include <string>
#include <sstream>
#include <iostream>

struct Point
{
    float x,y;
};

struct Triangle{
    int x, y ,z;
};


class ObjModel{
    private:
        std::vector<Point> Parrays; 
        std::vector<Triangle> Tarrays;
        std::string filePath;

  	public:
        ObjModel(std::string path);
        void show(); // 调试用
        std::vector<Point> getParrays(){return this->Parrays;};
        std::vector<Triangle> getTarrays(){return this->Tarrays;};

};
```



