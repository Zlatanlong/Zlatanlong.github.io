---
title: windows下找到linux的感觉
typora-copy-images-to: ./windows下找到linux的感觉
typora-root-url: ./windows下找到linux的感觉
date: 2020-09-10 16:21:35
updated:
categories:
  - windows
---

# 前言

对于开发人员来说，mac系统无疑是最受欢迎的。但是由于某些原因，比如游戏，娱乐，qiong，我们不想舍弃windows带来的方便，又不想双设备进行开发时，在windows下开发也就难免了。为了提升windows环境下的开发，找到linux带给我们便利的感觉。可以使用以下工具，找到开发的幸福感。

- WSL 适用于linux的windows子系统
- 升级到wsl2
- windows terminal
- zsh

<!--more-->

## 1. 安装WSl
WSL是在windows下进行开发，并使用linux中的bash等功能的最好的解决方案之一。

## 1.1 打开系统功能

**控制面板\所有控制面板项\程序和功能** 中选择 **启用或者关闭Windows功能**

![image-20200912150736512](/image-20200912150736512.png)

然后勾选 **适用于Linux的Windows子系统** 

![image-20200912150852217](/image-20200912150852217.png)

经过系统一段时间的自动下载后，重启计算机。

<!-- more -->

## 1.2 下载Ubuntu

### 1.2.1 下载

直接在**Microsoft store**中搜索**Ubuntu**，并启动即可。我使用的是18.04版本。

![image-20200912151114333](/image-20200912151114333.png)

### 1.2.2 设置新分发版

首次启动新安装的 Linux 分发版时，将打开一个控制台窗口，系统会要求你等待一分钟或两分钟，以便文件解压缩并存储到电脑上。 未来的所有启动时间应不到一秒。然后，需要为新的 Linux 分发版创建用户帐户和密码。

![Windows 控制台中的 Ubuntu 解包](https://docs.microsoft.com/zh-cn/windows/wsl/media/ubuntuinstall.png)

## 1.3 更新WSL到WSL2

[参考微软官方文档](https://docs.microsoft.com/zh-cn/windows/wsl/wsl2-index)

### 1.3.1 启用“虚拟机平台”可选组件

安装 WSL 2 之前，必须启用“虚拟机平台”可选功能。

以管理员身份打开 PowerShell 并运行：

PowerShell复制

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

**重新启动**计算机，以完成 WSL 安装并更新到 WSL 2。

### 1.3.2 将 WSL 2 设置为默认版本

以管理员的身份打开 PowerShell，然后在安装新的 Linux 发行版时运行以下命令，将 WSL 2 设置为默认版本：

PowerShell复制

```powershell
wsl --set-default-version 2
```

运行该命令后，你可能会看到此消息：`WSL 2 requires an update to its kernel component. For information please visit https://aka.ms/wsl2kernel`。 跟随链接（https://aka.ms/wsl2kernel），在文档中安装来自该页面的 MSI，以便在计算机上安装 Linux 内核供 WSL 2 使用。 安装内核后，请再次运行该命令，该命令应会成功完成而不显示消息。

 备注

从 WSL 1 更新到 WSL 2 可能需要几分钟才能完成，具体取决于目标分发版的大小。 如果从 Windows 10 周年更新或创意者更新运行 WSL 1 的旧（历史）安装，可能会遇到更新错误。 按照这些说明[卸载并删除任何旧分发](https://docs.microsoft.com/zh-cn/windows/wsl/install-legacy#uninstallingremoving-the-legacy-distro)。

如果 `wsl --set-default-version` 结果为无效命令，请输入 `wsl --help`。 如果 `--set-default-version` 未列出，则表示你的 OS 不支持它，你需要更新到版本 1903（内部版本 18362）或更高版本。



### 1.3.3 将分发版版本设置为 WSL 1 或 WSL 2

可打开 PowerShell 命令行并输入以下命令（仅在 [Windows 内部版本 18362 或更高版本](ms-settings:windowsupdate)中可用），检查分配给每个已安装的 Linux 分发版的 WSL 版本：`wsl -l -v`

```powershell
wsl --list --verbose
```

![image-20200912151921601](/image-20200912151921601.png)

若要将分发版设置为受某一 WSL 版本支持，请运行：

```powershell
wsl --set-version <distribution name> <versionNumber>
```

请确保将 `<distribution name>` 替换为你的分发版的实际名称，并将 `<versionNumber>` 替换为数字“1”或“2”。 可以随时更改回 WSL 1，方法是运行与上面相同的命令，但将“2”替换为“1”。

**我这里就是运行:**

```powershell
wsl --set-version Ubuntu-18.04 2
```

此外，如果要使 WSL 2 成为你的默认体系结构，可以通过此命令执行该操作：

```powershell
wsl --set-default-version 2
```

这会将安装的任何新分发版的版本设置为 WSL 2。

## 1.4 配置Ubuntu默认镜像源

### 1.4.1 备份并更新

```bash
cd /etc/apt  #进入配置文件所在目录
cp sources.list sources.list.bak  #备份配置文件
vim sources.list  #编辑配置文件
```

[参考阿里云Ubuntu镜像源配置方法](https://developer.aliyun.com/mirror/ubuntu?spm=a2c6h.13651102.0.0.3e221b11QxaLbF)

将对应的代码粘贴进去。

**ubuntu 18.04(bionic) 配置如下**:

```bash
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

这里如果在vim直接粘贴会出错。要不检查错误，还可以打开ubuntu的实际硬盘位置

`C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs`

这个就是 **Ubuntu** WSL版的根目录,接着到 **etc\apt** 下找到 **sources.list** 用之前贴的配置文件覆盖即可.接着更新数据源

### 1.4.2 更新了镜像源之后，更新apt和其他软件：

```bash
sudo apt-get update  #更新源  
sudo apt-get upgrade  #更新软件 
```

## 1.5 更改Ubuntu实际位置到d盘

[此步骤参考CSDN我的大神666Windows10子系统(WSL)修改安装目录](https://blog.csdn.net/lee_jackgg/article/details/106738878)

只要把ubuntu版本改成对应的即可。

### 1.5.1 首先查看所有分发版本

```powershell
wsl -l --all  -v
```

>   NAME STATE VERSION
>
>  * Ubuntu-20.04 Running 2

### 1.5.2 导出分发版为tar文件到d盘

```powershell
wsl --export Ubuntu-20.04 d:\ubuntu20.04.tar
```

##### 注销当前分发版

```powershell
wsl --unregister Ubuntu-20.04
```

### 1.5.3 重新导入并安装分发版在d:\ubuntu

```powershell
wsl --import Ubuntu-20.04 d:\ubuntu d:\ubuntu20.04.tar --version 2
```

### 1.5.4 设置默认登陆用户为安装时用户名

```powershell
ubuntu2004 config --default-user Username
```

### 1.5.5 删除tar文件(可选)

```powershell
del d:\ubuntu20.04.tar
```

## 2. Windows Terminal中使用WSL

### 2.1 安装

直接在Microsoft Store中搜索并安装即可。

![image-20200912153621967](/image-20200912153621967.png)

安装后应该可以识别到wsl，如果没有识别，可以在设置中进行如下配置：

```json
"list":
        [
            {
                "guid": "{c6eaf9f4-32a7-5fdc-b5cf-066e8a4b1e40}",
                "hidden": false,
                "name": "Ubuntu-18.04",
                "source": "Windows.Terminal.Wsl",
                "startingDirectory": "//wsl$/Ubuntu-18.04/root"
            },
            ...其他配置
        ]
```

这样就可以检测到wsl系统了，点击即可打开。

![image-20200912153808770](/image-20200912153808770.png)

### 2.2 配置zsh

```bash
~$ sudo apt-get install zsh #首先下载
/mnt/c/Users/User$ which zsh #查看下载是否成功
```

![image-20200912154357630](/image-20200912154357630.png)

如果可以出现如下目录，即为安装成功。

### 2.3 配置oh my zsh

根据[https://ohmyz.sh/](https://ohmyz.sh/)官网，可以输入命令：

```bash
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

但是我在安装时出现了错误：

```bash
curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused
```

经查证，需要配置ssh。这里我直接把windows下的**.ssh**文件，复制到了wsl文件下根目录**\\\wsl$\Ubuntu-18.04**的**root**文件里。

再次下载，成功。如果以前没有配置过github的ssh，我不知道是否还会报这个错误，如果抱着个错误了，请参考[如何生成ssh公私钥并将公钥放到github上](https://blog.csdn.net/u013778905/article/details/83501204)。

### 2.4 美化工作

```bash
cd ~ #进入root根目录
vim .zshrc #编辑.zshrc
```

将主题设置为：

ZSH_THEME="agnoster"

将windows terminal的默认字体设置为`Cascadia Code PL`，就是开启powerline功能的意思，agnoster的箭头需要开启此功能。

![image-20200912155336231](/image-20200912155336231.png)

并在下面schemes中添加如下配置：

```json
"schemes": [
        {
          "name": "Solarized Dark",
          "black": "#002831",
          "red": "#d11c24",
          "green": "#738a05",
          "yellow": "#a57706",
          "blue": "#2176c7",
          "purple": "#c61c6f",
          "cyan": "#259286",
          "white": "#eae3cb",
          "brightBlack": "#475b62",
          "brightRed": "#bd3613",
          "brightGreen": "#475b62",
          "brightYellow": "#536870",
          "brightBlue": "#708284",
          "brightPurple": "#5956ba",
          "brightCyan": "#819090",
          "brightWhite": "#fcf4dc",
          "background": "#001e27",
          "foreground": "#708284"
        },
        {
          "name": "Solarized Darcula",
          "black": "#25292a",
          "red": "#f24840",
          "green": "#629655",
          "yellow": "#b68800",
          "blue": "#2075c7",
          "purple": "#797fd4",
          "cyan": "#15968d",
          "white": "#d2d8d9",
          "brightBlack": "#25292a",
          "brightRed": "#f24840",
          "brightGreen": "#629655",
          "brightYellow": "#b68800",
          "brightBlue": "#2075c7",
          "brightPurple": "#797fd4",
          "brightCyan": "#15968d",
          "brightWhite": "#d2d8d9",
          "background": "#3d3f41",
          "foreground": "#d2d8d9"
        }
      ]
```

这些配置是在[iTerm2-Color-Schemes](https://github.com/mbadolato/iTerm2-Color-Schemes/tree/master/windowsterminal)中复制的，大家可以根据自己的喜好定制。并在对应的终端开启主题，这里我直接默认使用该主题了。

### 2.5 安装zsh插件

参考[一些常用的zsh插件](https://zhuanlan.zhihu.com/p/108636629)这篇文章，写的非常详细了。