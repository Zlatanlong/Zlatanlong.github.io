---
title: 使用nvm管理node版本
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
updated:
tags:
  - nvm
categories:
  - node
---
最近在处理一个两年前的后端项目时，npm跑不起来，根据报错是版本依赖问题，要不更改node版本，要不更新npm依赖，有一串依赖需要更新。担心更新了新的依赖后，源代码也需要更改，所以考虑直接使用mvn下载老版本的node，然后使用老版本的node运行。

<!-- more -->

1. [nvm-windows下载](https://github.com/coreybutler/nvm-windows/releases) nvm实际上只能在linux/mac系统上运行，如果windows系统需要下载这个nvm-setup.zip，安装时会自动配好环境变量。

2. 安装nvm 直接安装即可，第一个目录是以后个版本nodejs的安装目录，第二个目录实际上是第一个目录的快捷方式.

更改setting.txt文件，修改nodejs的镜像和npm的镜像,注意这里阿里淘宝镜像源已经都更新为**https**
```
root: D:\env\nvm
path: D:\env\nvm\nodejs
node_mirror: https://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/
```
3. 通过nvm安装node
```bash
nvm -v // 查看nvm版本
nvm install 8.12.0 // 下载指定版本 
nvm install latest安装最新版本
nvm use 8.12.0 // 使用指定版本
nvm ls // 查看已经安装的nodejs版本
node -v // 查看nodejs版本
```
4. 安装全局npm
　　安装node的时候，npm其实也已经一起安装了。因为nvm可以管理多个版本的node，如果每次添加一个node版本都要安装一堆的包很麻烦,如果有一个npm可以让各个版本的node共用，就不会这么麻烦了，这就是为什么我们要配置一个全局的npm的原因。简单的三步就可以配置一个全局的npm。
```bash
1.  npm config set prefix "D:\env\nvm"//配置用npm下载包时全局安装的包路径
2.  npm install npm -g//安装全局npm,不同的node都使用这个npm，想更新全局的npm的话首先删除全局路径(就是上一行命令的地址,可以使用npm config ls查看)下的npm,再执行一次这个命令即可
3.  在用户变量中添加 NPM_HOME=D:\env\nvm，path中添加%NPM-HOME%
```