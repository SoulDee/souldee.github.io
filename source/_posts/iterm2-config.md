---
title: iTerm2 终端配置
date: 2022-01-01 23:05:56
categories: Environment
tags:
   - macOS
   - terminal
---

最近因为工作的原因，需要配置 macOS 的环境，terminal 算是开发环境中非常常用的了。

<!-- more -->

放一张效果图 ![iTerm2 config](/images/iTerm2-config.png)

## Terminal

*[iTerm2](https://iTerm2.com/)* 下载打开即可安装

*[Oh My ZSH!](https://ohmyz.sh/)* 是 *zsh* 的配置管理工具，使用命令行安装

```bash
# curl
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# or

# wget , install wget before ohmyzsh if you using wget
brew install wget
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

## Theme

首先配置一个 zsh 的渲染主题（不是文字颜色主题），从 *[Theme wiki](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)* 中选择一个你喜欢的主题的名称，这里选择 *agnoster* ，然后配置主题。

```bash
# 1. Open config file
vim ~/.zshrc

# 2. find and change ZSH_THEME field.
ZSH_THEME="agnoster"
```

## Plugins

接着安装一些功能插件，从 *[Plugins wiki](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins)* 找到需要的插件下载，或者搜一下 *[zsh - Github](https://github.com/search?q=zsh)* 看一下，下面用两个插件示范。

```bash
# 1. Download plugins
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting

# 2. Open config file
vim ~/.zshrc

# 3. Find and change Plugins field.
plugins=(zsh-autosuggestions zsh-syntax-highlighting)

# 4. Add starter in the end
source ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
source ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
```

经过上面配置后再次打开 iTerm2 后就会有自动补全和代码高亮的效果。


## Font

字体在显示效果中也很重要。安装 *[Powerline - Github](https://github.com/powerline/fonts)* 

```bash
git clone https://github.com/powerline/fonts.git --depth 1
cd fonts
./install.sh
```

打开 *iTerm2* ，进入到文字设置标签 *Profiles>Open Profiles>Edit Profiles>Text* 找到 *font* 选项，选择名称后缀为 *for Powerline* 的任意字体。

将 *Text Rendering* 选项的 *Use build-in Powerline glyphs* 勾选上，不然会出现乱码。


## Window

适当的透明度和失焦能够带来很好的观感。进入到窗口标签 *Profiles>Open Profiles>Edit Profiles>Window*。

通过对属性 *Transparency* 的调整来修改窗口透明度，不需要太高，个人设置 20 左右。

通过对属性 *Blur* 的调整修改失焦，类似玻璃效果，个人设置 16 左右。

## Colors

最后我们还要设置一下各种文本的显示颜色，进入颜色标签 *Profiles>Open Profiles>Edit Profiles>Colors* 。

右下角 *Color Presets* 选择任意喜欢的主题，然后基于这个主题进行调整，*Basic Colors* 主要是基本颜色，右边的 *ANSI Colors* 则是各种状态的颜色，例如链接，命令，文件，补全文本等等，可以直接修改，iTerm2 会实时变更的。

如果你只想要更多的配色，而不想要自己慢慢调整，那么可以看看 *[iTerm2-Color-Schemes](https://github.com/mbadolato/iTerm2-Color-Schemes)* ，里面的主题不仅仅能用在 iTerm2 中。

```bash
#  Download iT er m-Color-Schemes
git clone https://github.com/mbadolato/iTerm2-Color-Schemes.git --depth 1
```

在项目中找到想要的主题的名字，例如 *3024 Day*

通过 *Profiles>Open Profiles>Edit Profiles>Colors* 中右下角 *Color Presets>Import* ,然后选择 *Term2-Color-Schemes>schemes>3024 Day.itermcolors* 文件。

或者也可以按住 *shift/command* 进行多选打开多个主题。

## Reference

1. [iTerm2](https://iTerm2.com/)
2. [Oh My ZSH!](https://ohmyz.sh/)
3. [onmyzsh wiki - Github](https://github.com/ohmyzsh/ohmyzsh/wiki)
4. [zsh-autosuggestions - Github](https://github.com/zsh-users/zsh-autosuggestions)
5. [zsh-syntax-highlighting - Github](https://github.com/zsh-users/zsh-syntax-highlighting.git)
6. [Powerline - Github](https://github.com/powerline/fonts)
7. [iTerm2-Color-Schemes - Github](https://github.com/mbadolato/iTerm2-Color-Schemes)