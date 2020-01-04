---
title: git基础
date: 2020-01-05 00:03:27
tags:
---

### git配置

Git 自带一个 `git config` 的工具来帮助设置控制 Git 外观和行为的配置变量。 这些变量存储在三个不同的位置：

1. `/etc/gitconfig` 文件: 包含系统上每一个用户及他们仓库的通用配置。 如果使用带有 `--system` 选项的 `git config` 时，它会从此文件读写配置变量。
2. `~/.gitconfig` 或 `~/.config/git/config` 文件：只针对当前用户。 可以传递 `--global` 选项让 Git 读写此文件。
3. 当前使用仓库的 Git 目录中的 `config` 文件（就是 `.git/config`）：针对该仓库。

### 用户信息

当安装完 Git 应该做的第一件事就是设置你的用户名称与邮件地址。 这样做很重要，因为每一个 Git 的提交都会使用这些信息，并且它会写入到你的每一次提交中，不可更改：

```console
$ git config --global user.name "NQiu"
$ git config --global user.email qiuwenlin@example.com
```

再次强调，如果使用了 `--global` 选项，那么该命令只需要运行一次，因为之后无论你在该系统上做任何事情， Git 都会使用那些信息。 当你想针对特定项目使用不同的用户名称与邮件地址时，可以在那个项目目录下运行没有 `--global` 选项的命令来配置。

### 文本编辑器

既然用户信息已经设置完毕，你可以配置默认文本编辑器了，当 Git 需要你输入信息时会调用它。 如果未配置，Git 会使用操作系统默认的文本编辑器，通常是 Vim。 如果你想使用不同的文本编辑器，例如 Emacs，可以这样做：

```console
$ git config --global core.editor vim
```

### 协议替换

当你想去克隆一个别人github上的repository时，发现系统不让你动，提示你防火墙禁止对git://的访问，这时候就只能用https://来访问repository。

```console
$ git config --global url."https://".insteadOf "git://"
$ git config --global --replace-all url."git@code.byted.org:".insteadOf "https://code.byted.org/"
```

### 别名

```console
$ git config --global alias.co checkout
```

### 检查配置信息

如果想要检查你的配置，可以使用 `git config --list` 命令来列出所有 Git 当时能找到的配置。

```console
$ git config --list
user.name=NQiu
user.email=qiuwenlin@example.com
color.status=auto
color.branch=auto
color.interactive=auto
color.diff=auto
...
```

你可能会看到重复的变量名，因为 Git 会从不同的文件中读取同一个配置（例如：`/etc/gitconfig` 与 `~/.gitconfig`）。 这种情况下，Git 会使用它找到的每一个变量的最后一个配置。

你可以通过输入 `git config <key>`： 来检查 Git 的某一项配置

```console
$ git config user.name
NQiu
```

### 获取帮助

若你使用 Git 时需要获取帮助，有三种方法可以找到 Git 命令的使用手册：

```console
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
```

### 获取仓库

有两种取得 Git 项目仓库的方法。 第一种是在现有项目或目录下导入所有文件到 Git 中； 第二种是从一个服务器克隆一个现有的 Git 仓库。

**现有目录中初始化仓库**

```console
$ cd <target-dir>
$ git init
```

**克隆现在仓库**

克隆仓库的命令格式是 `git clone [url]`。举例如下

```console
$ git clone https://github.com/nqiu/mydotfiles mydotfiles
```

### 记录每次更新到仓库

工作目录下的每一个文件都不外乎这两种状态：已跟踪或未跟踪。 已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后，它们的状态可能处于未修改，已修改或已放入暂存区。 工作目录中除已跟踪文件以外的所有其它文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有放入暂存区。 初次克隆某个仓库的时候，工作目录中的所有文件都属于已跟踪文件，并处于未修改状态。

编辑过某些文件之后，由于自上次提交后你对它们做了修改，Git 将它们标记为已修改文件。 我们逐步将这些修改过的文件放入暂存区，然后提交所有暂存了的修改，如此反复。

![文件的状态变化周期](/images/git-lifecycle.png)

### 常见命令一览

```console
$ git status
$ git add
$ git diff [--staged|--cached]
$ git commit [-a]
$ git rm [--cached]
$ git mv <old_file> <new_file>
$ git log [-p] [--author] [--pretty] [-(n)] [--stat] [--name-only] [--name-status] [--since|--after] [--util|--before] [--grep]
```

### 撤销操作

**修改已提交**

有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。 此时，可以运行带有 `--amend` 选项的提交命令尝试重新提交：

```console
$ git commit --amend
```

这个命令会将暂存区中的文件提交。 如果自上次提交以来你还未做任何修改（例如，在上次提交后马上执行了此命令），那么快照会保持不变，而你所修改的只是提交信息。

文本编辑器启动后，可以看到之前的提交信息。 编辑后保存会覆盖原来的提交信息。

例如，你提交后发现忘记了暂存某些需要的修改，可以像下面这样操作：

```console
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

最终你只会有一个提交——第二次提交将代替第一次提交的结果。

**撤销已暂存**

使用 `git reset HEAD <file>...` 来取消暂存。 所以，我们可以这样来取消暂存 `CONTRIBUTING.md` 文件：

```console
$ git reset HEAD CONTRIBUTING.md
Unstaged changes after reset:
M	CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

这个命令有点儿奇怪，但是起作用了。 `CONTRIBUTING.md` 文件已经是修改未暂存的状态了。

**撤销已修改**

未暂存区域是这样：

```console
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

它非常清楚地告诉了你如何撤消之前所做的修改。 让我们来按照提示执行：

```console
$ git checkout -- CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```

可以看到那些修改已经被撤消了。

### 远程仓库

管理远程仓库包括了解如何添加远程仓库、移除无效的远程仓库、管理不同的远程分支并定义它们是否被跟踪等等。

**查看远程仓库**

```console
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
```

**添加远程仓库**

运行 `git remote add <shortname> <url>` 添加一个新的远程 Git 仓库，同时指定一个你可以轻松引用的简写

```console
$ git remote add pb https://github.com/paulboone/ticgit
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)
```

**从远程仓库中抓取与拉取**

```console
$ git fetch [remote-name]
$ git pull [remote-name] [branch-name]
```

**推送到远程仓库**

```console
$ git push [remote-name] [branch-name]
```

**远程仓库的删、改、查**

```console
$ git remote show origin
$ git remote rename pb paul
$ git remote rm paul
```

### git打标签

Git 可以给历史中的某一个提交打上标签，以示重要。 比较有代表性的是人们会使用这个功能来标记发布结点。

```console
$ git tag
$ git tag -l 'v1.0.*'
```

Git 使用两种主要类型的标签：轻量标签（lightweight）与附注标签（annotated）。

一个轻量标签很像一个不会改变的分支——它只是一个特定提交的引用。

然而，附注标签是存储在 Git 数据库中的一个完整对象。 它们是可以被校验的；其中包含打标签者的名字、电子邮件地址、日期时间；还有一个标签信息；并且可以使用 GNU Privacy Guard （GPG）签名与验证。 通常建议创建附注标签，这样你可以拥有以上所有信息；但是如果你只是想用一个临时的标签，或者因为某些原因不想要保存那些信息，轻量标签也是可用的。

```console
# 创建附注标签
$ git tag -a v1.4 -m "my version 1.4"
# 创建轻量标签
$ git tag v1.8
# 查看tag详细信息
$ git show v1.4
# 补打tag，附注标签
$ git tag -a v1.2 9fceb02
# 共享tag
$ git push origin [tagname]
$ git push origin --tags
# 删除本地累量标签
$ git tag -d v1.8
# 使用git push <remote> :refs/tags/<tagname>更新远程标签
$ git push origin :refs/tags/v1.8
# 检出标签，分离头指针状态detached HEAD，修改将不属于任何分支
$ git checkout v1.2
# 检出标签，新分支改动后会向前移动，与v1.2会有不同
$ git checkout -b version2 v1.2
```

