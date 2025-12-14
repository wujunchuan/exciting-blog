---
title: M1 Mac 环境配置记录
published: 2021-02-04
description: '新电脑环境配置过程记录，Apple silicon YES!'
image: 'https://storage.fuckwechat.com/apple_silicon_1493_yyyy12Su_171956.webp'
tags: ['macOS']
category: '开发环境'
draft: false
lang: 'zh'
---

# 前言

正常情况下我会在更换电脑时用 `Time Machine` 直接恢复原有的环境。距离我上次搭建环境已经过去 4 年了，想要趁着这次换电脑的契机彻底重新搭建一遍。

:::note
建议定期使用 `Time Machine` 对机器进行定期备份。
:::

以下是配置记录：

## 必备软件

### XCode

:::tip
一般也不用这个步骤，因为安装 `Homebrew` 的时候，就会自动安装。
:::

> 简单来讲，Command Line Tools 就是一个小型独立包，为 Mac 终端用户提供了许多常用的工具、实用程序和编译器。包括 svn、git、make、GCC、clang、perl、size、strip、strings、libtool、cpp、what 以及许多能够在 Linux 默认安装中找到的有用命令。

如果没有 `Xcode IDE` 的需求，可以直接只安装 `Xcode Command Line Tools` 版本：

```bash
xcode-select --install
```

### Homebrew

参考文档：[M1 芯片 Mac 上 Homebrew 安装教程](https://juejin.cn/post/6914139996299460615)

#### 导出 bundle 列表

在旧电脑导出使用 `Homebrew` 安装的软件包列表：

```bash
/usr/local/bin/brew bundle dump
```

:::tip
这里我选择不直接导入 `Brewfile` 文件，而是选择缺少的包进行安装。
如果有需要导入 `Brewfile` 文件的话，可以执行 `brew bundle --file="~/Brewfile"`，具体 API 可以查阅相关文档。
:::

#### 多版本共存

~~鉴于当前 `Homebrew` 还没有完成对 M1 芯片的适配，因此我们需要安装两个版本的 `Homebrew`，并通过别名 `alias` 使两个版本共存。~~

:::warning
已经将两个版本合并，不需要单独安装两个版本了。
:::

##### 安装不同版本的 `Homebrew`

安装 ARM 版本：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

~~安装 X86 版本：`arch -x86_64 /bin/bash -c "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install.sh)"`~~

~~设置别名到对应的配置文件中：~~

```bash
~~alias abrew='arch -arm64 /opt/homebrew/bin/brew'
alias ibrew='arch -x86_64 /usr/local/bin/brew'~~
```

#### 日常更新依赖

```bash
#!/bin/bash
brew update -v && brew upgrade -v && brew cleanup
```

#### 加入环境变量

执行完安装脚本后，注意看输出信息。执行 `echo 'eval $(/opt/homebrew/bin/brew shellenv)' >> $HOME/.zprofile` 后重启终端，将 `Homebrew` 加入环境变量。

:::tip
zsh 配置文件的加载顺序：
`.zshenv` → `.zprofile` → `.zshrc` → `.zlogin` → `.zlogout`

将耗时长，并且不经常变动的配置信息放到 `~/.zprofile` 中，这样大部分的耗时只发生在打开 Terminal 时，从而提高执行 `source ~/.zshrc` 的速度。
:::

### Node.js

多版本管理工具：[nvm](https://github.com/nvm-sh/nvm)

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash
```

:::warning
🚧 当前 `Node.js` 版本为 15.3.0 以上的版本才有提供 `ARM64` 版本。

查看 `Node.js` 运行的架构信息，执行 `node -p process.arch`
:::

参考这个 [issue](https://github.com/nvm-sh/nvm/issues/2350#issuecomment-734132550) 可以知道，安装 `v15` 版本以下的版本，需要在 `Rosetta2` 的加持下。

> For anything under v15, you will need to install node using Rosetta 2. In Terminal, use `arch -x86_64 zsh`

当前官方提供的 v15 只有源码版本，需要下载到本地编译安装，因此安装时间会比较久。

![](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/713c3be7-08db-466d-9871-792c612a9e00/Untitled.png)

:::warning
建议管理多版本 `Node.js` 时，全局 npm 与全局 yarn 包分别安装管理。即不要存在跨版本的任何依赖。
:::

:::tip
💡 根据 `.nvmrc` 自动切换环境
:::

```bash
# Location: ~/.zshrc
# Calling nvm use automatically in a directory with a .nvmrc file
# place this after nvm initialization!
autoload -U add-zsh-hook
load-nvmrc() {
  local node_version="$(nvm version)"
  local nvmrc_path="$(nvm_find_nvmrc)"

  if [ -n "$nvmrc_path" ]; then
    local nvmrc_node_version=$(nvm version "$(cat "${nvmrc_path}")")

    if [ "$nvmrc_node_version" = "N/A" ]; then
      nvm install
    elif [ "$nvmrc_node_version" != "$node_version" ]; then
      nvm use
    fi
  elif [ "$node_version" != "$(nvm version default)" ]; then
    echo "Reverting to nvm default version"
    nvm use default
  fi
}
add-zsh-hook chpwd load-nvmrc
load-nvmrc
```

### Visual Studio Code

当前（2021年02月04日）只有 [insider](https://code.visualstudio.com/insiders/) 版本才原生支持 M1 芯片。强烈建议使用 insider 版本，性能提升非常明显。

:::tip
💡 使用官方提供的 [Settings Sync](https://code.visualstudio.com/docs/editor/settings-sync) 特性，同步到新电脑的时候记得选择 insider 版本，这样就能把旧配置同步到 insider 版本了。
:::

## 终端命令行工具

### oh-my-zsh

[oh-my-zsh](https://ohmyz.sh/#install) 是一个社区驱动的框架，用于管理 zsh 配置。

安装命令：

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### oh-my-tmux

[oh-my-tmux](https://github.com/gpakosz/.tmux) 是一个让 Tmux 更易使用的配置集合。

安装步骤：

```bash
cd # 进入 Home 目录
git clone https://github.com/gpakosz/.tmux.git # 克隆仓库
ln -s -f .tmux/.tmux.conf
cp .tmux/.tmux.conf.local .
```

### fzf

fzf 是一个用 `golang` 编写的通用命令行模糊搜索工具。

### smartmontools

M1 电脑过度读写 SSD 问题排查（查看硬盘读写统计）

:::note
一些用户表示，他们发现 M1 在读写数据时会过度使用 Mac 电脑的 SSD，这一问题可能会影响到 M1 版 Mac 中搭载的 SSD 的寿命，进而影响整个机器的寿命。
:::

使用步骤：

1. 安装：`brew install smartmontools`
2. 执行 `diskutil list`，确定自己 SSD 所在盘符，默认一般为 `disk0`
3. `smartctl -a disk0`

参考：[How to check your Intel and M1 Mac's SSD health using Terminal](https://www.macworld.com/article/3609512/how-to-m1-intel-mac-ssd-health-terminal-smartmontools.html)
