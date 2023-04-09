---
title: Windows Terminal 配置
tags:
  - Windows Terminal
  - Theme
  - Font
abbrlink: '11e1167'
date: 2023-03-31 12:07:08
---

早在 Windows Terminal 刚出来没多久有简单试用了一下，但是因为~~懒惰~~没空折腾，然后在前几天，本来我只是在从 npm 迁移到 pnpm 之后，因为觉得 pnpm 太长并且都是右手感觉手难受，于是设置一下 alias，但是不知道为什么始终只在 Windows Terminal 生效，而在 VSCode 里面的终端不生效（原因在下面），于是就顺手折腾了一下，顺便记录一些遇到的问题。（为什么别人安装那么顺利而我按照他们顺序就是有报错啊啊啊啊啊啊！！！）

<!-- more -->

## Windows Terminal

首先当然是安装 Windows Terminal 了，系统自带的 Microsoft store 直接搜索并安装即可，或者访问[网页版](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701?hl=zh-cn&gl=cn&rtc=1)跳转也行（因为之前不知道为什么把商店关了导致搜不到）。

Windows Terminal 是一个终端容器，以及提供了设置项的可视化面板，实际还是调用的其他终端。

通过上方横幅的向下的三角箭头可以打开终端设置，可以在这里进行可视化的修改，也可以通过左下角的 `settings.json` 来修改。

```JSON
{
    ...
    "profiles": {
        // 默认配置
        "defaults": {...},
        // 配置列表
        "list": [
            {
                ...
                "name": "PowerShell 7",
                ...
            },
        ]
    },
    // 文字配色方案列表
    "schemes": [
        {
            ...
            "name": "3024 Day",
            ...
        },
    ]
}
```

## PowerShell 7.x

在这个 [Release](https://github.com/PowerShell/PowerShell/releases) 页面根据需求选择安装包下载并安装。 

然后左下角菜单中找到 PowerShell 7，右键-以管理员身份打开。这里只是先创建 profile 文件并保存，后续才会用到。

```bash
# 后面统一使用 code 打开不在赘述

# 1. 获取路径
echo $profile # 一般是 C:\Users\username\Documents\PowerShell\Microsoft.PowerShell_profile.ps1

# 2. 打开路径
notepad $profile # 使用系统自带笔记本打开
code $profile # 或者使用 VSCode 打开

# 3. 直接保存到 1 的那个路径里
```
PS：系统自带了一个 Windows PowerShell，两者有关但并不等同，可以在系统中同时存在，具体可以参考 [Windows PowerShell 5.1 与 PowerShell 7.x 之间的差异](https://learn.microsoft.com/zh-cn/powershell/scripting/whats-new/differences-from-windows-powershell?view=powershell-7.2)。

两者启动的程序以及启动配置的 `profile.ps1` 文件是不同的路径，我在开头提到的问题就是因为不清楚这一点，因此我在 Windows Terminal 中设置了 Windows PowerShell 的 `profile.ps1` 文件，但是在 VSCode 中配置的终端是 PowerShell 7.x。

Windows PowerShell，启动的程序是 `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`， PowerShell 7.x。启动的程序是 `C:\Program Files\PowerShell\7\pwsh.exe`，对应的配置文件一般是 exe 同根目录下创建的 `profile.ps1`，或者根据官网文档 [PowerShell 7.4 about_Profiles](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7.4)、[Windows PowerShell about_Profiles](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-5.1) 选择其他限定范围。

接下来打开 Windows Terminal 设置，找到 *添加新配置文件*，修改名称：PowerShell 7，命令行：`"C:\Program Files\PowerShell\7\pwsh.exe"`，图标：`ms-appx:///ProfileIcons/pwsh.png`，其他默认就行。

保存之后你应该就可以通过上方 `+` 来选择并打开 PowerShell 了。

## scoop(可选)

[scoop](https://scoop.sh/) 是命令行的程序安装工具，我们用来安装 `Oh My Posh` 的，也可以使用 `winget`，或者 PowerShell 自带的 `cmdlet`，但是这两个有一些问题，`winget` 我在系统商店安装之后依然没法使用，使用 github 的安装包安装会提示被限制安装失败。而 `cmdlet` 貌似有兼容性问题，并且需要手动设置环境变量。

打开 PowerShell 通过以下命令安装 `scoop`:

```bash
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser # Optional: Needed to run a remote script the first time
irm get.scoop.sh | iex
```

## On My Post

>Oh My Posh is a custom prompt engine for any shell that has the ability to adjust the prompt string with a function or variable.

简单说可以让你有输入提示，并且可以配置主题，字体等内容。在 PowerShell 通过以下命令安装。

```bash
Set-ExecutionPolicy Bypass -Scope Process -Force; Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://ohmyposh.dev/install.ps1'))
```

安装结束之后复制以下内容并粘贴到刚刚创建的配置文件当中，主题可以在 [Themes](https://ohmyposh.dev/docs/themes) 选择，替换名称即可。profile 文件会在终端启动后加载执行，否则我们就需要每次都手动设置一次才可以使用。

```bash
# 通过修改 themeName 来切换主题
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\themeName.omp.json" | Invoke-Expression
```

此时重启 PowerShell 你应该就可以看到对应主题内容了，并且对于你曾经输过的命令，你只需要输入前几个字母，后面会有浅色气体提示，你按一下右方向键就会自动填充。

PS：部分主题使用了字体图标，因此会导致乱码，解决办法就是下一步的字体安装，但也有一些主题没有用到字体图标，那么使用任意字体都行。

## Font(可选)

`On My Post` 的许多主题都是用了字体图标来显得更加美观，为了使这些带图标的主题可用，就需要安装 [Nert Font](https://www.nerdfonts.com/font-downloads)，在 `On My Post` 的文档中是使用命令行安装，但在我这里不知道什么原因，不管挂没挂梯子都会超时，因此推荐在 [font-download](https://www.nerdfonts.com/font-downloads) 直接下载压缩包。

可以在 [programming fonts](https://www.programmingfonts.org/) 预览这些字体的实际效果。压缩包解压会看到一堆的 `ttf` 字体文件，加粗、斜体等等，选择带有 `Nert Font` 的其中一个安装即可。

打开 Windows Terminal 设置，选择配置 PowerShell，拉到下面的 *外观*，然后修改字体为刚刚安装的字体保存后马上生效。

PS：如果不使用带图标主题，可以选择任意字体，或者自行安装其他字体文件。

## Font Color(可选)

配色方案是用来修改字体颜色的，我们可以基于 Windows Terminal 提供的默认方案自行修改，或者使用现成的方案。

[Windows Terminal Themes](https://windowsterminalthemes.dev/) 当中有许多的配色方案可选，你可以预览后选择一个选择 'Get theme'，这会将配色方案复制到剪贴板。

打开 Windows Terminal 设置，左下角点击 *打开 JSON 文件*，此时打开的是 `settings.json`，将上面已经复制的配色方案粘贴到 `schemes` 的数组中。

或者拉到下面的 `Download .json of themes` 来下载全部配色，然后全部粘贴到 `schemes` 当中。接着就可以在外观中选择这些配色方案了。

## gsudo(可选)

Windows Terminal 默认是没有管理员权限，为了解决这个问题可以使用 [gsudo](https://gerardog.github.io/gsudo/)，通过以下命令安装：

```bash
scoop install gsudo
```

然后打开 Windows Terminal 设置-添加新配置文件-复制配置文件-下拉选择PowerShell-点击复制。然后将名字修改为 `PowerShell Plus` 或者其他有所区分的名称，然后修改该配置的命令行为：`gsudo.exe pwsh.exe -nologo`。

这时候你就能通过下拉菜单的 `PowerShell Plus` 来打开管理员权限的终端，也可以将该配置设为默认终端。此时标签名称为 `Administrator：PowerShell Plus` 表示管理员权限。

## VSCode(可选)

作为个人常用 IDE，自然也有必要添加配置。打开文件-首选项-配置文件(默认)-显示内容，然后选择左边的 `settings.json` 打开，添加下面内容。

```json
{
    "terminal.integrated.profiles.windows": {
        // 添加 PowerShell 配置项
        "PowerShell": {
        "source": "PowerShell",
        "icon": "terminal-powershell"
        }
    },

    // 设置默认终端为 PowerShell
    "terminal.integrated.defaultProfile.windows": "PowerShell",
}
```

此时打开终端应该可以看到和 Windows Terminal 相同效果。

## Other

Windows Terminal 本身还有其他内置外观修改可以自行探索：

- 复古风格终端效果
- 光标形状
- 【强烈推荐使用】毛玻璃特效(enable acrylic) 和不透明度(建议 0.7-0.9)。
- 背景图

## Reference

1. [Microsoft store](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701?hl=zh-cn&gl=cn&rtc=1)
2. [PowerShell Release](https://github.com/PowerShell/PowerShell/releases)
3. [Windows PowerShell 5.1 与 PowerShell 7.x 之间的差异](https://learn.microsoft.com/zh-cn/powershell/scripting/whats-new/differences-from-windows-powershell?view=powershell-7.2)
4. [PowerShell 7.4 about_Profiles](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7.4)
5. [Windows PowerShell about_Profiles](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-5.1)
6. [scoop](https://scoop.sh/)
7. [oh my posh - Themes](https://ohmyposh.dev/docs/themes)
8. [Nert Font](https://www.nerdfonts.com/font-downloads)
9. [programming fonts](https://www.programmingfonts.org/)
10. [Windows Terminal Themes](https://windowsterminalthemes.dev/)
11. [gsudo](https://gerardog.github.io/gsudo/)