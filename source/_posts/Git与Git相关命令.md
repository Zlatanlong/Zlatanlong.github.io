---

title: Git与Git相关命令
date: 2020-03-17 12:41:03
typora-copy-images-to: ./Git与Git相关命令
typora-root-url: Git与Git相关命令
tags:
  - Git
categories:
  - Git
---

# Git 常用命令

> [Pro Git 中文版](http://iissnan.com/progit/html/zh/ch2_6.html)

## 最常用

`git status`

`git add .`

`git commit -m "balabala"`

git commit -amend 将此次更新文件并入到上次commit的记录中，不新添加commit

```bash
git commit --amend
```

如果上一次提交没提交完，这个命令可以把当前文件添加到上一次提交中。

git push`

<!-- more -->

## 删除子模块

`git ls-tree HEAD`

![ls-tree](/6EF187FF-E93A-41B6-B244-3A4C508078F6.png)

其中 commit 说明这一行这个文件夹被git当成子模块了。如果想转变回当前git的模块直接删除其.git还不行，

还要用下面的命令 git rm --cached MyBatisPlus_page_tables

这样它就从你的git tree中移除了

再用 git add. commit添加后 发现就是当前git树的内容了。

`git rm --cached -r .`

关于  git rm --help 中的--cached参数

--cached
           Use this option to unstage and remove paths only from the index. Working tree files, whether modified or not, will be left alone.

## 初始化

`git init`

`git remote add origin git@github.com:User/project.git`

## 分支相关

`git checkout -b newbranch`

`git merge anotherbranch`

git branch //显示本地分支
git branch -a //显示远程分支

## ssh连接远程相关

`ssh-keygen -t rsa -C “user@hostname.com”`

`ssh-add privateKey`

`ssh-add -K privateKey`