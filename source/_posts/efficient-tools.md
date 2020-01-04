---
title: 效率工具
date: 2019-12-28 15:57:06
tags:
---

工欲善其事，必先利其器。本文介绍我日常工作用到的命令行工具，包括zsh, tmux, vim等工具、常用配置、快捷键，以及一些小技巧。

## 写在前面 ##

1. 没有最好的配置，只有最适合自己的配置
2. 一种好的配置需要不断练习，让它成为一种习惯，相信一万小时定律
3. 站在巨人的肩膀上，不断地吸纳好的工作方式，提高工作效率

## terminal ##

mac上首选[iTerm2](iterm2.com "iTerm2")，它是一个MacOS上完全免费的终端工具，支持分屏等功能；

对于Linux系统中有类似的终端工具Terminator。

## shell ##

Linix系统默认自带的shell是bash，推荐使用zsh，zsh兼容bash语法。配合[ohmyzsh](https://github.com/ohmyzsh/ohmyzsh "ohmyzsh")使用，有各式各样的主题和插件可选择，让你在命令行操作6得飞起。

1. zsh安装

   ```shell
   ubuntu$ sudo apt install zsh
   mac$ brew install zsh
   ```

   将zsh做为默认的登录shell

   ```shell
   ubuntu$ sudo chsh -s /bin/zsh
   注: macos高版本默认为zsh，低版本请自行搜索
   ```

2. 安装ohmyzsh

   ```shell
   sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
   ```

   or

   ```shell
   sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
   ```

3. 安装好看的主题

   ```shell
   # ~/.zshrc
   ZSH_THEME="random"
   ZSH_THEME_RANDOM_CANDIDATES=(
       "robbyrussell" "agnoster" "ys" "sunrise" "avit" "kolo"
       "pygmalion" "muse" "bureau" "blinks" "bira" "af-magic"
       "sorin" "igeek" "enlightenment" "spaceship" "lambda-mod"
       "elessar" "pi" "lambda-gitster"
   )
   # 注：以上配置了我喜欢的主题，有些主题需要手动安装，以下示范安装lambda-mod主题
   $ git clone https://github.com/halfo/lambda-mod-zsh-theme "${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/lambda-mod"
   $ ln -s "${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/lambda-mod/lambda-mod.zsh-theme" "${ZSH:-~/.oh-my-zsh}/themes/lambda-mod.zsh-theme"
   ```

4. 安装插件

   a. 安装常用的zsh插件

   ```shell
   # zsh-autosuggestions
   ubunut$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
   # zsh-syntax-highlighting
   ubuntu$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
   # history-substring-search
   ubuntu$ git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search
   # zsh-completions
   ubuntu$ git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:=~/.oh-my-zsh/custom}/plugins/zsh-completions
   # git-open
   ubuntu$ git clone https://github.com/paulirish/git-open.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/git-open
   # asdf
   ubuntu$ git clone https://github.com/asdf-vm/asdf.git $HOME/.asdf
   # autoenv
   ubuntu$ git clone git://github.com/inishchith/autoenv.git ~/.autoenv
   # fzf
   ubuntu$ git clone --depth 1 https://github.com/junegunn/fzf.git $HOME/.fzf
   ubuntu$ $HOME/.fzf/install --key-bindings --completion --no-update-rc --no-bash --no-fish
   ```

   b. 配置.zshrc文件

   ```shell
   # ~/.zshrc
   plugins=(
       zsh-autosuggestions zsh-syntax-highlighting history-substring-search zsh-completions asdf tmux
       git git-flow z urltools extract themes history encode64 docker web-search colored-man-pages
       git-open emoji-clock mosh autoenv thefuck fzf
   )
   ```

## tmux ##

tmux(terminal multiplexer)是一款丰常强大的终端复用工具。tmux启动后，它创建一个**session**及一个**window**，并展示到屏幕，屏幕下方有一个状态栏展示当前session信息，并可用来输入交互命令。tmux使用**session**和**window**来管理多个**pseudo terminal**。一个**session**可以拥有一个或多个**window**，当前活动**window**占据整个屏幕，并可以被分割成多个**panel**，每个**panel**是一个独立的**pseudo terminal**。

1. 安装tmux

   ```shell
   # 版本不低于2.2
   ubuntu$ sudo apt install tmux
   ```

2. 安装oh-my-tmux

   ```shell
   ubuntu$ cd
   ubuntu$ git clone https://github.com/gpakosz/.tmux.git
   ubuntu$ ln -s -f .tmux/.tmux.conf
   ubuntu$ cp .tmux/.tmux.conf.local .
   ```

## vim ##

vim是Linix下强大的文本编辑，网络有许多优秀的配置集合，我最终选择了spf13-vim作为我的配置。

1. 安装vim

   ```shell
   ubuntu$ sudo apt install vim
   ```

2. 安装spf13-vim

   ```shell
   ubuntu$ curl https://j.mp/spf13-vim3 -L > $VIM_BASE_DIR/spf13-vim.sh && sh $VIM_BASE_DIR/spf13-vim.sh
   ```
