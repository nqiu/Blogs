---
title: git原理
date: 2020-01-05 01:56:48
tags:
---

## 写在前面

 前面介绍了一系列的git命令来玩转git，它还包含了一部分用于完成底层工作的命令。 这些命令被设计成能以 UNIX 命令行的风格连接在一起，抑或藉由脚本调用，来完成工作。这部分命令一般被称作“底层（plumbing）”命令，而那些更友好的命令则被称作“高层（porcelain）”命令。

底层命令有助于说明 Git 是如何完成工作的，以及它为何如此运作。 多数底层命令并不面向最终用户：它们更适合作为新命令和自定义脚本的组成部分。在介绍原理之前，先介绍几个基本概念和用到的底层命令，以便更轻松理解下面的内容。

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
   * hash-object
   * cat-file
   * update-index

## 参考

* [Wiki - Git FAQ]("https://git.wiki.kernel.org/index.php/GitFaq" "Git FAQ")
* [Git from the inside out]("https://codewords.recurse.com/issues/two/git-from-the-inside-out" "Git from the inside out")