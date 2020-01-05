---
title: git原理
date: 2020-01-05 01:56:48
tags:
---

## 写在前面

 前面介绍了一系列的git命令来玩转git，它还包含了一部分用于完成底层工作的命令。 这些命令被设计成能以 UNIX 命令行的风格连接在一起，抑或藉由脚本调用，来完成工作。这部分命令一般被称作“底层（plumbing）”命令，而那些更友好的命令则被称作“高层（porcelain）”命令。

底层命令有助于说明 Git 是如何完成工作的，以及它为何如此运作。 多数底层命令并不面向最终用户：它们更适合作为新命令和自定义脚本的组成部分。在介绍原理之前，先介绍几个基本概念和用到的底层命令，以便轻松理解下面的内容。

* 三种状态
   * Untracked，还未git add，对应工作目录(Working Directory)
   * Staged，git add后的状态，对应暂存区(Staging Area)
   * Committed，git commit后的状态，对应版本库(.git directory, Repository)
* git对象
   * blob, 文件内容快照
   * tree, 代表文件/目录结构，包含权限、blob对象指针、文件名等信息
   * commit, 代表一次提交，包含tree对象指针、本次提交的parent指针、作者等信息
   * tag, 标签，包含commit对象指针、tag附加信息等
* 底层命令
   * hash-object, 根据文件创建blob对象，并返回object-id
   * cat-file, 查看git对象的内容、类型和大小等
   * update-index, 将工作空间的文件内容登记到index文件
   * write-tree, 根据index，创建tree对象
   * commit-tree, 创建commit对象

### 揭开.git的神秘面纱

当在一个新目录或已有目录执行 `git init` 时，Git 会创建一个 `.git` 目录。 这个目录包含了几乎所有 Git 存储和操作的对象。  该目录的结构如下所示：

```console
$ ls -F1
HEAD
config
description
hooks/
info/
objects/
refs/
```

该目录下可能还会包含其他文件，不过对于一个全新的 `git init` 版本库，这将是你看到的默认结构。 `description` 文件仅供 GitWeb 程序使用，我们无需关心。 `config` 文件包含项目特有的配置选项。 `info` 目录包含一个全局性排除（global exclude）文件，用以放置那些不希望被记录在 .gitignore 文件中的忽略模式（ignored patterns）。 `hooks` 目录包含客户端或服务端的钩子脚本（hook scripts）。

剩下的四个条目很重要：`HEAD` 文件、（尚待创建的）`index` 文件，和 `objects` 目录、`refs` 目录。 这些条目是 Git 的核心组成部分。 `objects` 目录存储所有数据内容；`refs` 目录存储指向数据（分支）的提交对象的指针；`HEAD` 文件指示目前被检出的分支；`index` 文件保存暂存区信息。 

#### 演示1（使用高层命令）

```console
$ pwd  # 目录1
$ mkdir test && cd test
$ git init
$ tree .git
.git
├── HEAD
├── config
├── description
├── hooks
│   ├── pre-commit.sample
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
$ echo "Hello World" > file.txt
$ git add file.txt
$ tree .git
.git
├── HEAD
├── config
├── description
├── hooks
│   ├── pre-commit.sample
├── index
├── info
│   └── exclude
├── objects
│   ├── 55
│   │   └── 7db03de997c86a4a028e1ebd3a1ceb225be238
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
$ vi .git/index
Head: master

Staged (1)
A file.txt
$ git commit -m 'initial commit: hello world'
$ tree .git
.git
├── COMMIT_EDITMSG
├── HEAD
├── config
├── description
├── hooks
│   ├── pre-commit.sample
├── index
├── info
│   └── exclude
├── logs
│   ├── HEAD
│   └── refs
│       └── heads
│           └── master
├── objects
│   ├── 55
│   │   └── 7db03de997c86a4a028e1ebd3a1ceb225be238
│   ├── 58
│   │   └── ea66451e34e7b6554f6c844d7c7d243f5b14bb
│   ├── da
│   │   └── 26f9458b3e4ec925acbb0b0e86a21fae388d5f
│   ├── info
│   └── pack
└── refs
    ├── heads
    │   └── master
    └── tags
```

#### 演示2（使用底层命令）

```console
$ pwd  # 目录2，注意演示1和演示2在不同目录进行，以方便我们对比
$ mkdir test && cd test
$ tree .git
.git
├── HEAD
├── config
├── description
├── hooks
│   ├── pre-commit.sample
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
$ echo "Hello World" > file.txt
$ git hash-object -w file.txt
557db03de997c86a4a028e1ebd3a1ceb225be238
$ git update-index --add --cacheinfo 100644 \
  557db03de997c86a4a028e1ebd3a1ceb225be238 file.txt
# 或者使用 git update-index --add file.txt
$ vi .git/index
$ git write-tree
da26f9458b3e4ec925acbb0b0e86a21fae388d5
$ tree .git
.git
├── HEAD
├── config
├── description
├── hooks
│   ├── pre-commit.sample
├── index
├── info
│   └── exclude
├── objects
│   ├── 55
│   │   └── 7db03de997c86a4a028e1ebd3a1ceb225be238
│   ├── da
│   │   └── 26f9458b3e4ec925acbb0b0e86a21fae388d5f
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
$ echo 'initial commit: hello world' | git commit-tree da26f9
e85ddcf3291ef71c11656cceeffcf31ae428903d
$ tree .git
.git
├── HEAD
├── config
├── description
├── hooks
│   ├── pre-commit.sample
├── index
├── info
│   └── exclude
├── objects
│   ├── 55
│   │   └── 7db03de997c86a4a028e1ebd3a1ceb225be238
│   ├── da
│   │   └── 26f9458b3e4ec925acbb0b0e86a21fae388d5f
│   ├── e8
│   │   └── 5ddcf3291ef71c11656cceeffcf31ae428903d
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
```

### git对象

Git 是一个内容寻址文件系统，Git 的核心部分是一个简单的键值对数据库（key-value data store）。 你可以向该数据库插入任意类型的内容，它会返回一个键值，通过该键值可以在任意时刻再次检索（retrieve）该内容。

对比**演示1**和**演示2**，我们发现objects目录（即git对象数据库）结构和文件名称是完全一致的。目录是以底层命令返回的Object-ID（SHA-1哈希值, 40位）的前2位为目录名，后38位为文件名组织的。下面我们使用`git cat-file`工具查看一个各哈希值存储什么样的内容。

**blob对象**, 即`git add`或`git hash-object`生成的object-id

```console
# 查看对象内容
$ git cat-file -p 557db03de997c86a4a028e1ebd3a1ceb225be238
Hello World
# 查看对象大小
$ git cat-file -s 557db03de997c86a4a028e1ebd3a1ceb225be238
12
# 查看对象类型
$ git cat-file -t 557db03de997c86a4a028e1ebd3a1ceb225be238
blob
```

**tree对象**, 即`git write-tree`后生成的object-id

```console
# 查看对象内容
$ git cat-file -p da26f9458b3e4ec925acbb0b0e86a21fae388d5
100644 blob 557db03de997c86a4a028e1ebd3a1ceb225be238    file.txt
# 查看对象大小
$ git cat-file -s da26f9458b3e4ec925acbb0b0e86a21fae388d5
36
# 查看对象类型
$ git cat-file -t da26f9458b3e4ec925acbb0b0e86a21fae388d5
tree
```

**commit对象**, 即`git commit-tree`生成的object-id

```console
# 查看对象内容
$ git cat-file -p e85ddcf3291ef71c11656cceeffcf31ae428903d
tree da26f9458b3e4ec925acbb0b0e86a21fae388d5f
author nqiu <867217317@qq.com> 1578196100 +0800
committer nqiu <867217317@qq.com> 1578196100 +0800

initial commit: hello world
# 注: 首次提交只包含tree/author/committer/message信息，以后提交还包含parent信息

# 查看对象大小
$ git cat-file -s e85ddcf3291ef71c11656cceeffcf31ae428903d
174
# 查看对象类型
$ git cat-file -t e85ddcf3291ef71c11656cceeffcf31ae428903d
commit
```

### 工作目录和分支

工作目录是一次提交/分支的检出，其中`.git/HEAD`文件记录了commit-id或分支名称，表示当前工作目录中是哪个提交或分支的检出

```console
$ cat .git/HEAD
ref: refs/heads/master
```

我们再看`.git/refs`目录，其中heads目录记录了分支名称（文件名），文件内容为当前分支所指向的commit-id。

```console
$ cat .git/refs/heads/master
58ea66451e34e7b6554f6c844d7c7d243f5b14bb
```

这样我们很容易手动创建一个分支，只需将某个commit-id写入.git/refs/heads目录即可

```console
$ echo '58ea66451e34e7b6554f6c844d7c7d243f5b14bb' > .git/refs/heads/br-1
$ git branch
* master
  br-1
```

我们也可以手动将当前工作目录替换为另一个分支的内容

```console
$ echo "ref: refs/heads/br-1" > .git/HEAD
```



## 参考

* [Wiki - Git FAQ]("https://git.wiki.kernel.org/index.php/GitFaq" "Git FAQ")
* [Git from the inside out]("https://codewords.recurse.com/issues/two/git-from-the-inside-out" "Git from the inside out")