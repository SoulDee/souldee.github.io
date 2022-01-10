---
title: Qt 多平台打包记录
date: 2021-12-28 00:10:44
categories: [Environment,Qt]
tags: [Qt,C++]
---

本文记录在 Qt 跨平台应用开发完成之后的打包步骤，每个平台都列出了 Qt 版本和系统环境，因为这些都会对打包过程有一定的影响，例如 macOS 默认下载 xcode 的 SDK 版本过新，而 Qt 5.15.2 支持不了等，会在下面写出为什么选择这些版本，希望能给你带来帮助。

<!-- more -->

## Windows

Windows 下打包应该是最容易的，直接只用 Qt 提供的工具运行即可

### 环境

- Windows 10
- Qt 5.15.2



### 打包


1. 环境变量配置，Windows 的环境变量不多说了，需要配置的是下面这样的路径
```bash
C:\Qt\5.15.2\5.15.2\msvc2019_64\bin
```
2. 假设应用名称为 TestApp，Qt Creator 中 Realease 模式运行一次
3. 找到 Realease 路径中同名应用复制到 TestApp 文件夹
4. 终端运行下方内容
```bash
windeployqt TestApp
```
4. 对于最终生成的代码可以进行一些适当的删除
```bash
# 这几个在 Widget 开发中基本都有用到
Qt5Widgets.dll # widgets 模块，QML 应用可以删除
Qt5Gui # Gui 基本是要的
Qt5Core.dll # Core 模块
platforms/qwindows.dll # 平台相关插件
styles # Qt 样式表，没用到自定义样式表的话可以删除
translations # 国际化内容，没用到也可以删除

# 下面如果代码中没用到都可以删
iconengines/qsvgicon.dll # Svg
imageformats # 各种图片格式的支持，默认png 已经支持，可以删除
Qt5Svg.dll # svg
opengl32sw.dll # OpenGL
D3Dcompiler_47.dll # OpenGL
libEGL.dl # OpenGL
libGLESv2.dll # OpenGL

```

5. 启动 TestApp.exe 直接可用，然后将全部压缩成压缩包发给其他人吧。

### 静态文件

windows 下不想打包到 exe 的静态文件，可以在运行打包后直接放入根目录


## macOS
macOS 比较特殊的一点是芯片分为了 intel 架构和 m1 芯片的 arm 架构，需要分别打包，而 Qt 6.2 版本才支持 arm 架构，打包步骤没什么差别。此外要 Qt 5.14.2 仅支持 xcode sdk 10.15，所以选择 Qt 6.2.1，也可以选择将 xcode 降级。

### 环境

- macOS Monterey 12.0.1 arm | macOS Big SUr 11.1 intel
- Qt 6.2.1



### 打包

1. 需要先商店安装 xcode，不然会缺少 kits
2. 【可选】macOS 虚拟机卡顿严重，可以参考链接 9 优化一下
3. 配置 macdeployqt 命令
```bash
sudo vim ~/.zshrc

# 插入以下内容后保存
export QT_DEPLOY_PATH=/Users/你的用户名/Qt/6.2.1/macos/bin
export PATH=$QT_DEPLOY_PATH:$PATH

# 使生效
source ~/.zshrc

# 验证生效，看下有没有刚加入的路径
echo $PATH 
```
4. 假设应用名称 TestApp，在Qt Creator 中 Release 模式运行一次
5. 找到 Release 路径，拷贝 TestApp 到同名文件夹
6. 移动到同名文件夹打开终端
```bash
macdeployqt TestApp.app -dmg
```
7. 将打包好的 dmg 文件发给别人吧

### 静态文件

macOS 下静态文件，需要在运行 release 后，右键 app 文件显示包内容，然后放进 Contents/MacOs 里。

## Linux

linux 下打包会比较麻烦，需要说明的是，Ubuntu 的版本和 Qt 的版本都会影响环境配置，linuxdeployqt 这个 打包工具当前仅支持到 Ubuntu 18.04，基于这些原因选择当前系统版本和 Qt 版本。

### 环境

- VMware Workstation Pro
- Ubuntu 18.04 x64
- Qt 5.14.2

### 安装

1. 安装包 [qt-opensource-linux-x64-5.14.2.run](https://mirrors.tuna.tsinghua.edu.cn/qt/archive/qt/5.14/5.14.2/qt-opensource-linux-x64-5.14.2.run)，如果不能拖拽需要通过 **虚拟机-安装 VMware Tools **来安装相关工具。
2. 然后双击安装，选择组件时记得选择 Qt5.14.2/Desktop gcc 64-bit
3. 【可选】更换 Ubuntu 的apt 镜像源为 [阿里云ubuntu 镜像](https://developer.aliyun.com/mirror/ubuntu?spm=a2c6h.13651102.0.0.3e221b11ZMu5Np) ，具体更换步骤点进去查看，注意系统版本。下面方便使用复制了 18.04 的源。
```bash
sudo vim /etc/apt/sources.list

# 将这个文件内容替换为以下内容
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
4. 在终端中安装环境包
```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install build-essential
sudo apt-get install libqt4-dev

```
5. 创建一个项目试试看吧



### 打包 linuxdeployqt

1. 下载打包工具 [linuxdeployqt-continuous-x86_64.AppImage](https://github.com/probonopd/linuxdeployqt/releases/download/continuous/linuxdeployqt-continuous-x86_64.AppImage)

2. 配置 linuxdeployqt 命令

```bash
# 重命名
mv linuxdeployqt-continuous-x86_64.AppImage linuxdeployqt

# 移动到用户的 bin 目录，就可以全局调用了
sudo mv linuxdeployqt /usr/local/bin

# 测试一下可用性
linuxdeployqt --version
```

3. 配置Qt 环境变量

```bash
# 将下面的[gcc 目录]替换为你得 Qt 安装目录下的 gcc 路径

# 使用 sudo 打开 ~/.bashrc 并在末尾插入以下内容
export QT_BIN_PATH=[gcc 目录]/bin
export QT_LIB_PATH=[gcc 目录]/lib
export QT_PLUGINS_PATH=[gcc 目录]/plugins
export PATH=$QT_BIN_PATH:$QT_LIB_PATH:$QT_PLUGINS_PATH:$PATH

# 使配置生效并验证可用性，如果输入的 $PATH 有你刚刚的路径则成功
source ~/.bashrc
echo $PATH
```

4. 生成 Release 包，假设应用为 TestApp

5. Release 模式运行一次，在左下角切换模式。
6. 进入到 Release 路径，路径可以查看侧边栏 Project 中 'Build Settings'-General-'Build directory' 的路径。
7. 在 Release 中找到TestApp 并将它拷贝到一个新建的TestApp文同名件夹中
8. 在该文件夹中创建一个 default.desktop 文件夹和你要作为图标的图片，使用任意编辑器打开 default.desktop 写入下面内容，见下方
```
[Desktop Entry]
Type=Application
Name=[app 打包后名称]
Exec=AppRun %F
Icon=图标名称
Comment=一些描述
Categories=Utility;
```
9. 【可选】添加 **libqgtk2 & libqgtk2style** ，避免一些打包错误
```bash
# if Error message
WARNING: Plugin "/usr/lib/x86_64-linux-gnu/qt4/plugins/platformthemes/libqgtk2.so" not found, skipping
WARNING: Plugin "/usr/lib/x86_64-linux-gnu/qt4/plugins/styles/libqgtk2style.so" not found, skipping

# Fix 如果没有安装 git 请先执行 sudo apt-get install git 来安装
apt-get update
apt-get -y install libgtk2.0-dev
git clone http://code.qt.io/qt/qtstyleplugins.git
cd qtstyleplugins
qmake
make -j$(nproc)
sudo make install
cd -
```
10. 打开终端运行打包 ，这样我们得到打包后的 .AppImage 文件了，将它发送给其他人使用吧
```bash
linuxdeployqt TestApp -appimage
```

### 打包 Qt Installer Framework

详情请看 [How to Deploy Your Qt Cross-Platform Applications to Linux Operating System With the Qt Installer Framework - medium](https://medium.com/geekculture/how-to-deploy-your-qt-cross-platform-applications-to-linux-operating-system-with-qt-installer-ac28258bc370)

### 静态文件

linux 下静态文件，就不能打包成 AppImage，而是要将相关库文件复制过来，如果是使用 linuxdeployqt 进行打包，只需要修改打包命令的参数。

```bash
linuxdeployqt TestApp -always-overwrite
```

## 参考链接

1. [清华大学开源镜像站](https://mirrors.tuna.tsinghua.edu.cn/qt/)
2. [Categories= key missing in Linux desktop file · Issue #1914 · balena-io/etcher](https://github.com/balena-io/etcher/issues/1914)
3. [ldd not finding libqgtk2 platform theme.](https://github.com/probonopd/linuxdeployqt/issues/350)
4. [在Linux下使用linuxdeployqt发布Qt程序](https://www.cnblogs.com/linuxAndMcu/p/11016322.html)
5. [How to Deploy Your Qt Cross-Platform Applications to Linux Operating System With linuxdeployqt -medium](https://medium.com/swlh/how-to-deploy-your-qt-applications-to-linux-operating-system-with-linuxdeployqt-3c004a43c67a)
6. [ubuntu16.0.4（linux）安装qt5.9.4步骤、配置gcc和启动](https://blog.csdn.net/naibozhuan3744/article/details/84328872)
7. [ubuntu 安装g++](https://blog.csdn.net/klarclm/article/details/8550931)
8. [QT使用windeployqt部署发布及其精简](https://blog.csdn.net/itas109/article/details/80497065)
9. [虚拟机卡顿_三步解决Mac虚拟机、黑苹果 卡顿的问题，图文教程送工具](https://blog.csdn.net/weixin_35437233/article/details/112577708)

