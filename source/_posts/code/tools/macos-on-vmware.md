---
title: 📓 VMware 下安装 macOS 踩坑
date: 2021-12-12 19:21:55
tags: 
    - VMware
    - macOS
---

更新：以下内容有待验证，例如后期验证过 VMware 16.2 也是可以用于安装 macOS 的。

有时候我们需要使用 macOS 的一些东西，但是又不想专门买一个笔记本电脑，例如使用 Qt 开发跨平台的桌面应用，我们只需要使用 macOS 来编译对应版本的运行包而已。 这时候虚拟机就派上用场了。关于安装的步骤请直接看底部参考链接1，这里主要记录安装过程遇到的一些问题和解决方案。个人建议先浏览一下在去看参考链接1，特别是如果你也是 AMD 的 CPU 。如果你是 Inter 的 N 卡，可能问题没这么多。

<!-- more -->


### ⚙ 相关硬件参数

CPU：AMD Ryzen 5 3600



### ✅ VMware workstation pro 的选择和安装

选择 15.0 或者 15.1 版本，序列号随便搜一下安装教程一般最下面都有，以下是原因：

- 16 因为目前找到的用于解锁安装 macOS 系统的 Unlocker 最多支持到 15
- 15.5 如果使用的是 AMD 的 CPU 可能是不行的，会导致一个 **未能启动虚拟机** 的问题
- 15.1 未实测，但是根据一些搜索结果来看 AMD 也可以
- 15.0 实测可行



### ✅ Unlocker 的选择和使用

选择 paolo-projects/**[unlocker](https://github.com/paolo-projects/unlocker)**，可以下载 [Release](https://github.com/paolo-projects/unlocker/releases/tag/3.0.3) 的打包版本或者自行安装 Python 2.7 的环境来直接运行。

- theJaxon/**[unlocker](https://github.com/theJaxon/unlocker)** 是早期版本，作者很久没有更新了，亲测失效，比较坑的是你在 Github 搜 unlocker 找到的第一个就是这个
- paolo-projects/**[unlocker](https://github.com/paolo-projects/unlocker)** 是上面那个的 fork 分支，亲测可用，但是 Github 你搜索 unlocker 的时候前面几页都没看到，倒是能看到同作者的另一个基于 C++ 写的 auto-unlocker，或许也可以？

下载之后以管理员权限运行目录下的 win_install.cmd 并等待执行完毕即可



### ✅ macOS 的镜像

如果你每次开启虚拟机后并没有进入macOS的加载画面，而是进入到蓝屏的 boot manager 的话，那么很可能是因为你镜像有问题，可以找一些别人已经试过的镜像。比如参考链接1中的两个镜像。

- dmg 是原版系统，虚拟机用不了，如果要使用的话需要一个大于 8g 的 U 盘制作启动盘，不推荐这个方法。个人使用 UltraISO 去制作启动盘，出现了一个新的问题，制作后的启动盘不能被虚拟机识别，可能虚拟机版本有关，测试用的 15.5，另外写入镜像后剩余的磁盘是未分配状态，所以你插入启动盘电脑能找到，但是在 **个人电脑** 是看不到的，需要到磁盘管理手动分配空间。
- ISO/CDR 系统镜像，虚拟机可用，如果有 dmg 又不想下载别人发的镜像也可以尝试自己将 dmg 转换为 ISO/CDR 格式，参考链接2，除此之外，cdr 格式在WMware 中会说可能不可用，但是其实是可以的，直接改一些后缀为 iso 即可。



### ✅ 客户机操作系统已禁用 CPU，请关闭或重置虚拟机

点击你的虚拟机，然后右键选择打开虚拟机目录，使用任意文本编辑器打开 **【你的虚拟机名称】.vmx** 这样的一个文件，添加以下内容保存并重新打开虚拟机

> ```
> smc.version = "0"
> cpuid.0.eax = "0000:0000:0000:0000:0000:0000:0000:1011"
> cpuid.0.ebx = "0111:0101:0110:1110:0110:0101:0100:0111"
> cpuid.0.ecx = "0110:1100:0110:0101:0111:0100:0110:1110"
> cpuid.0.edx = "0100:1001:0110:0101:0110:1110:0110:1001"
> cpuid.1.eax = "0000:0000:0000:0001:0000:0110:0111:0001"
> cpuid.1.ebx = "0000:0010:0000:0001:0000:1000:0000:0000"
> cpuid.1.ecx = "1000:0010:1001:1000:0010:0010:0000:0011"
> cpuid.1.edx = "0000:1111:1010:1011:1111:1011:1111:1111"
> featureCompat.enable = "FALSE"
> ```



### ✅ **这个虚拟机需要avx2，但是没有avx**

打开 vmx 文件，然后找到 virtualHW.version = "xx" ，将 xx 修改为 10，这个值代表的是兼容到哪个版本的 WVware。





### ✅ 启动后鼠标键盘无法移动

打开 vmx 文件，添加以下内容并修改虚拟机硬件配置中的 USB 配置器兼容性为 2.0，同时勾选 **显示所有 USB 设备**

> ```
> keyboard.vusb.enable = "TRUE"
> mouse.vusb.enable = "TRUE"
> ```



### ✅ 无法安装 VMware Tool

需要使用 darwin.iso 来安装，早期版本的 VMware 是自带的，后面一些版本没有，只能自己下载了，可以看一下参考链接 5，里面包含了下载链接。 

1. 安装前先将桌面右上角的磁盘退出，右键选中推出”Install macOS xxxx“。

2. 下载后修改虚拟机硬件配置 CD/DVD 镜像为 darwin.iso 的路径，并勾选设备状态为已连接就可以正常安装了。

3. 安装结束之后重启，然后从主机随便拖拽一个文件到虚拟机，这时候会让你进行 **安全性与隐私** 偏好设置，设置一下就行。



### ✅ 无法全屏

如果是 windows 和 linux ，在安装 VMware Tool 之后通常就自动铺满虚拟机了，而 macOS 麻烦点，并且需要虚拟机全屏才能触发 macOS 全屏。

1. 首先按照上面步骤安装 VMware Tool
2. 打开 vmx 文件并添加 bios.forceSetupOnce = "TRUE" 来进入 bios
3. 在 boot manager 界面中一次选择 Enter setip -> Boot from a file -> Recovery 【一串很长的名字】 -> 【一串序列号】-> boot.efi
4. 进入到一个和安装时一样的界面，点击上面工具栏的实用工具->终端，输入 **csrutil disable** 来关闭 macOS 的 SIP 保护服务
5. 之后输入 reboot 重启，并点击虚拟机上方工具栏的全屏模式，需要注意的是虚拟机退出全屏后 macOS 全屏也会失效。



### 📚 参考

1. [全网最详细的VMware虚拟机安装MacOS系统教程，没有之一！！！附全部资源 - xuan yang的文章 - 知乎](https://zhuanlan.zhihu.com/p/337036027) 
2. [黑苹果 dmg,cdr和iso的区别](https://blog.csdn.net/qq_31683775/article/details/120779678)
3. [AMD Vmware15 装 MaCOSX 10.14 报错# 客户机操作系统已禁用 CPU，请关闭或重置虚拟机](https://blog.csdn.net/silentbird520/article/details/96039415)
4. [macos虚拟机鼠标不能移动和键盘不能使用](https://blog.csdn.net/sunshine__sun/article/details/114674695)
5. [darwin.iso](https://crifan.github.io/popular_virtual_machine_vmware/website/usage_summary/vmware_tools/macos/darwin_iso.html)
6. [虚拟机苹果macOS系统安装VMware Tools教程](https://blog.csdn.net/weixin_40448001/article/details/116779835?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.highlightwordscore&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.highlightwordscore)
7. [详解：MacOS全屏显示，VMware Tools的安装与使用](https://blog.csdn.net/cait_/article/details/89505022)

