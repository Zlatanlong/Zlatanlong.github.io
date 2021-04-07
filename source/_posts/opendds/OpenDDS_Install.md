---
title: OpenDDS 安装并配置Java
typora-copy-images-to: ./img
typora-root-url: ./img
categories:
  - OpenDDS
---

# OpenDDS 安装并配置Java

本安装文档根据OpenDDS 3.16 的指导手册以及源码下README总结Windows下的安装方式

## 一、环境要求

- VS2017 / **VS2019** 作为OpenDDS的项目生成
- Perl5.32.1  作为OpenDDS示例的运行脚本（On Windows we recommend the use of [Straweberry Perl](https://strawberryperl.com).）
- jdk1.5以上 推荐使用1.8  实际上在安装过程中只用到了**%JAVA_HOME%\include\jni.h**与**%JAVA_HOME%\include\%JAVA_PLATFORM%\jni_md.h**两个文件。
- [OpenDDS 3.16](https://opendds.org/downloads.html)
- 离线环境需要：[ACE/TAO 2.2a patch 19 or later](http://download.objectcomputing.com/TAO-2.2a_patches/?M=D)

## 二、安装步骤

### 1. 首先解压源码：

假设OpenDDS解压到**E:\OpenDDS-3.16**

如果是离线环境，还需要将**ACE/TAO 2.2a patch 19**压缩包下的文件夹**ACE_wrappers**

### 2. 生成解决方案

启动**VS的开发人员命令行**，定位到**E:\OpenDDS-3.16**下，输入 **configure** 命令（如果要开启**Java**支持，需要使用 **configure --java**），等待命令执行完毕

此时程序自动生成了**DDS_TAOv2.sln**文件和**setenv.cmd**文件，在命令行执行**setenv.cmd**设置环境变量，然后使用`devenv DDS_TAOv2.sln`命令打开新生成的**sln**文件，这样可以保留环境变量、重定项目目标，然后编译源码并等待操作完成。如果是**64**位系统，也需要编译为**64**位版本（把Win32换成**x64**）

### 3. 配置环境变量

新建环境变量： 

**DDS_ROOT = E:\OpenDDS-3.16** 

**ACE_ROOT = E:\OpenDDS-3.16\ACE_wrappers** 

并将

- %DDS_ROOT%\bin
- %DDS_ROOT%\lib
- %ACE_ROOT%\bin
- %ACE_ROOT%\lib

添加到**path**变量中。

### 4. 运行测试

OpenDDS自带了很多实例，存放在**%DDS_ROOT%\examples和%DDS_ROOT%\tests\DCPS**下（**Java**示例在**%DDS_ROOT%\java\tests**下）

随便挑一个输入**perl run_test.pl**运行perl脚本，能正常收发数据即可

### 注意：

- Java实例会有很多JNI的输出，长得和异常信息差不多，但实际上没有出错，只要最后提示**test passed**就是成功的
- 第三步编译OpenDDS时候经常会出现找不到文件的错误，大多数和环境变量有关系，configure后生成的解决方案对文件的链接出错，在命令行中执行**setenv.cmd**设置环境变量，然后使用`devenv DDS_**.sln`命令打开新生成的**sln**文件
- 可能会提示缺少 jni_md.h，该文件位于 **%JAVA_HOME%\include\win32** 目录下，复制到 **%OPENDDS_HOME%\java\idl2jni\runtime** 目录下即可

