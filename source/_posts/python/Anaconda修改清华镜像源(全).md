---
title: Anaconda修改清华镜像源(全)
typora-copy-images-to: ./Anaconda修改清华镜像源(全)
typora-root-url: ./Anaconda修改清华镜像源(全)
date: 2020-10-21 16:21:35
categories:
  - Python
---

# 全部镜像源

> 参考[Anaconda 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)，配置还是要看官网呀。

大家在安装anaconda之后都会配置镜像源，但是网上好多教程都只是配置

**但这是不够的**。

以下为半错配置。

```
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
```

刚才在安装pytorch时候发现速度很慢。

<!--more-->

![img](/TG0Z0{3NQI7`T$IXJ@RIQUT.png)

回看之前的提示，pytorch的本体的源并不是清华的。

下载速度感人。

![img](/suduganren.png)

直接去清华镜像源网站上看看吧。

![image-20201021152433721](/image-20201021152433721.png)

```
channels:
  - defaults
show_channel_urls: true
channel_alias: https://mirrors.tuna.tsinghua.edu.cn/anaconda
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```

问题是这个channels的配置文件保存在哪那？各系统都可以通过修改用户目录下的 `.condarc` 文件。Windows 用户无法直接创建名为 `.condarc` 的文件，可先执行 `conda config --set show_channel_urls yes` 生成该文件之后再修改。

- Windows：`‪C:\Users\{你的用户名}\.condarc`

- Linux/Mac：`~/.condarc`

说白了还是用户文件下，编辑这个文件将内容粘贴即可。

接着根据清华镜像源的说明。

- 运行 `conda clean -i` 清除索引缓存，保证用的是镜像站提供的索引。

- 运行 `conda create -n myenv numpy` 测试一下吧。