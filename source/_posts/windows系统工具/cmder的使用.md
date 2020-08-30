---
title: cmder的使用
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
updated:
tags:
  - cmder
categories:
  - windows
---
更换了windows电脑，感觉有点开倒车，安装各种娱乐软件确实很爽，但是一到写代码各种环境又开始麻烦了。

终端没有什么好的方案，wsl感觉需要学习成本，那么这里直接用cmder吧。感觉界面还可以。

1. 下载cmder，在[cmder官网](https://cmder.net/)下载就可以，因为已经有git了，所以下载min版本即可。
2. 因为是win版本，所以要进行一些基本配置。一个是环境变量，一个是在任意位置右键出现`Cmder Here`这个功能。
   1. 在系统环境变量的path中增加Cmd.exe的路径（路径指它所在的文件夹）
   2. 以管理员身份打开cmd，执行以下命令即可，完了以后在任意地方点击右键即可使用cmder
    ```bash
    // 设置任意地方鼠标右键启动Cmder
    Cmder.exe /REGISTER ALL
    ```
3. vscode的配置，参考[cmder github上的说明](https://github.com/cmderdev/cmder/wiki/Seamless-VS-Code-Integration#use-cmder-embedded-git-in-vscode),在setting.json加入。
    ```
    "terminal.integrated.shell.windows": "C:\\Windows\\System32\\cmd.exe",
    "terminal.integrated.shellArgs.windows": ["/k", "D:\\cmder\\vendor\\init.bat"],
    ```
    这个方式实际上是先打开windows的cmd，然后再通过`cmd \k D:\\cmder\\vendor\\init.bat`这个命令打开cmder。曲线救国。
4. JetBrains家族软件的配置，这个等我用到了再整理。。。