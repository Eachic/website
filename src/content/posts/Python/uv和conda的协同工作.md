---
title: UV和conda的协同工作
published: 2026-04-17
tags: [Python,Pytoch]
category: Python
image: 'https://t.alcy.cc/moez'
draft: false
---


# UV和conda的协同工作


## 痛点分析
uv 几乎已经是一个比较完美的python 项目管理加包管理工具了，用起来还是非常方便的，  
但是，在实际使用过程中发现还是很难完全抛弃conda，主要是涉及到深度学习相关的库的时候（主要是pytorch）  
因为pytorch 的版本的特殊性，同一个版本号存在cpu版本和gpu版本，并且在多数时候都会默认安装cpu版本。  这就非常恶心了。  
因此如果涉及到深度学习库，主要是和学术研究有关并不需要很强的项目管理时，conda还是很有用的。  
但是conda 和 uv 同时存在与主环境感觉会冲突，因此我选择的环境管理为，conda 依旧是全局安装，然后单独给在conda 中开启一个环境，就命名为uv_env 这个环境只用来装uv

## 实际操作

首先在环境中装好conda(推荐miniconda)  
然后创建新的虚拟环境
```
conda create -n uv_env python=3.12
```
切换到对应虚拟环境后用pip下载uv
```
conda activate uv_env
pip install uv
```

这样还有一个好处，uv 下载的缓存包会存在对应虚拟环境中所在文件夹中（**注意并不是在虚拟环境中**），这样可以不用占c盘

## uv工作流

我们这里以部署开发版nanobot为例
首先 切换到uv_env文件夹
```
conda activate uv_env
```

然后进入到nanobot文件夹后执行
```
uv sync 
```
uv sync 会创建一个新的虚拟环境，我们可以使用``.venv/Scripts/activate``
也可以直接关闭终端，再进入，vscode一般是可以识别到当前目录下的虚拟环境的  
然后使用``nanobot onboard`` 启动即可


## conda 工作流

首先创建一个新的环境
```
conda creat -n nanobot_env python=3.14
```
然后切换到对应环境中后，安装
```
conda activate nanobot_env
pip install -e .
```
最后进入到对应的虚拟环境中
执行 ``nanobot onboard``

## uv 安装对应版本的pytorch

conda 一直是pytorch 的最佳安装器，但是很多需要严格项目管理的项目也需要用到pytorch 
其实uv 也是可以实现合理版本的pytorch的安装的，而且还可以实现不同平台安装不同的pytorch。主要依靠pyproject.toml 实现

实现方式如下：
```
[tool.uv.sources]
torch = [
  { index = "pytorch-cpu", marker = "sys_platform != 'linux'" },
  { index = "pytorch-cu128", marker = "sys_platform == 'linux'" },
]
torchvision = [
  { index = "pytorch-cpu", marker = "sys_platform != 'linux'" },
  { index = "pytorch-cu128", marker = "sys_platform == 'linux'" },
]

[[tool.uv.index]]
name = "pytorch-cpu"
url = "https://download.pytorch.org/whl/cpu"
explicit = true

[[tool.uv.index]]
name = "pytorch-cu128"
url = "https://download.pytorch.org/whl/cu128"
explicit = true
```