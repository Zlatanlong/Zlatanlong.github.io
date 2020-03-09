---
title: Mac系统工具文件命令大全(持续更新)
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
date: 2019-05-03 11:20:20
updated:
tags:
  - mac工具
  - mac命令
  - mac快捷键
categories:
  - mac
---

## 工具类

### homebrew

配置brew镜像：

```bash
# 1. 替换brew.git:

cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

# 2. 替换homebrew-core.git:

cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

# 3. 重置brew.git:

cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git

# 4. 重置homebrew-core.git:

cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core.git
```



<!-- more -->

### vscode

扩展使用：可以把非公共插件全部禁用，然后根据项目在扩展禁用栏中选择启动至工作区，这样即可以按需引入插件。

使用`Setting Sync`插件，将配置同步至github gist中，在切换设备时完美找回之前的配置

`git lens`无与伦比的git插件。

### 命令行

iTerm2

zsh

oh-my-zsh

.zshrc  tree配置

```bash
source ~/.bash_profile
alias tree="find . -print | sed -e 's;[^/]*/;|____;g;s;____|; |;g'"
PATH=/bin:/usr/bin:/usr/local/bin:${PATH}
export PATH
```

### Alfred4

### 其他：

#### gif图制作：igif builder

#### 截图：jietu

这个工具是Tencent集成到mac版qq中的，其中保存到粘贴板，可以直接用来粘贴到typora中写博客，并且图片质量刚刚好。

![image-20200306230921579](/assets/image-20200306230921579.png)

#### ftp/sftp传输： filezilla

#### 视频播放：mpv

#### 数据库GUI：Sequel Pro (开发版本)

开发版本解决了sequel在高版本macos中每次关闭报错。

#### 接口测试：postman

#### 系统管理：lemon

Tencent推出的系统清理管理工具lemon，你问我为什么不用cleanmymacX ？lemon免费

#### magnet pro

这是一款控制窗口大小位置的好用app

## 常用快捷键

😎呼出emoji选择器：(control + command + space)

切换全屏应用：control + ←/→

调度中心：control + ↑

finder快速goto：command + shift + g

## 常用文件位置

- host: `/etc/hosts`

- ~/.bash_profile: `/Users/zlatan`

~代表你的/home/用户明目录

假设你的用户名是x，那么~/就是/home/x/

.是代表此目录本身，但是一般可以不写

所以cd ~/. 和cd ~ 和cd ~/效果是一样的

但是.后面有东西又是另外一个问题，点在文件名头部，代表一个隐藏文件

~/.local是你的主目录下一个.local的文件夹的路径，

并且从.可以看出，这是一个饮藏文件，

如果不用ls -a的话，一般ls是无法看到的

- xcode命令行工具：`/Library/Developer/CommandLineTools/`

  ```bash
  xcode-select install
  
  xcode-select --switch /Library/Developer/CommandLineTools
  ```


## 常用命令

`chsh`: Linux chsh命令用于更改使用者 shell 设定。

使用权限：所有使用者。

## 常见文件名

`文件名带rc`

在Linux中，最为常用的缩略语也许是“rc”，它是“runcomm”的缩写――即名词“run command”(运行命令)的简写。rc是任何脚本类文件的后缀，这些脚本通常在程序的启动阶段被调用，通常是Linux系统启动时。如/etc/rc（连接到/etc/rc.d/rc）是Linux启动的主脚本，而.bashrc是当Linux的bash shell启动后所运行的脚本。

` .bashrc`

的前缀“.”是一个命名标准，它被设计用来在用户文件中隐藏那些用户指定的特殊文件;“ls”命令默认情况下不会列出此类文件，“rm”默认情况下也不会删除它们。许多程序在启动时，都需要“rc”后缀的初始文件或配置文件，这对于Unix的文件系统视图来说，没有什么神秘的。

