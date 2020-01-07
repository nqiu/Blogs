---
title: git分支
date: 2020-01-05 14:00:49
tags:
---

## 回顾

### git对象

我们假设现在有一个工作目录，里面包含了三个将要被暂存和提交的文件。 暂存操作会为每一个文件计算校验和（ SHA-1 哈希算法），然后会把当前版本的文件快照保存到 Git 仓库中（Git 使用 blob 对象来保存它们），最终将校验和加入到暂存区域等待提交：

```console
$ git add README test.rb LICENSE
$ git commit -m 'The initial commit of my project'
```

当使用 `git commit` 进行提交操作时，Git 会先计算每一个子目录（本例中只有项目根目录）的校验和，然后在 Git 仓库中这些校验和保存为树对象。 随后，Git 便会创建一个提交对象，它除了包含上面提到的那些信息外，还包含指向这个树对象（项目根目录）的指针。如此一来，Git 就可以在需要的时候重现此次保存的快照。

现在，Git 仓库中有五个对象：三个 blob 对象（保存着文件快照）、一个树对象（记录着目录结构和 blob 对象索引）以及一个提交对象（包含着指向前述树对象的指针和所有提交信息）。

![首次提交对象及其树结构](/images/commit-and-tree.png)

做些修改后再次提交，那么这次产生的提交对象会包含一个指向上次提交对象（父对象）的指针。

![提交对象及其父对象](/images/commits-and-parents.png)

### 分支

Git 的分支，其实本质上仅仅是指向提交对象的可变指针。 Git 的默认分支名字是 `master`。 在多次提交操作之后，你其实已经有一个指向最后那个提交对象的 `master` 分支。 它会在每次的提交操作中自动向前移动。

![](/images/branch-and-history.png)

## 分支管理

### 创建分支

```bash
$ git branch testing
```

![两个指向相同提交历史的分支](/images/two-branches.png)

现在有两个分支了，那么当前是在哪个分支呢？HEAD是一个特殊的指针，它指向当前所在分支。

![HEAD 指向当前所在的分支](/images/head-to-master.png)

### 切换分支

```bash
$ git checkout testing
```

![HEAD 指向当前所在的分支](/images/head-to-testing.png)

### 分支合并

在分支上做一些提交

```bash
$ vi test.rb
$ git commit -am 'made a change'
```

![HEAD 分支随着提交操作自动向前移动](/images/advance-testing.png)

 `testing` 分支向前移动了，但是 `master` 分支却没有，它仍然指向运行 `git checkout` 时所指的对象。我们切换回 `master` 分支

```bash
$ git checkout master
```

![](/images/checkout-master.png)

这条命令做了两件事。 一是使 HEAD 指回 `master` 分支，二是将工作目录恢复成 `master` 分支所指向的快照内容。 也就是说，你现在做修改的话，项目将始于一个较旧的版本。 本质上来讲，这就是忽略 `testing` 分支所做的修改，以便于向另一个方向进行开发。

在master分支，做一些修改

```bash
$ vi test.rb
$ git commit -am 'made another change'
```

![](/images/advance-master.png)

**FastForward(快进)合并**

![](/images/basic-branching-4.png)

![](/images/basic-branching-5.png)

**3-way Merge三方合并**

![](/images/basic-branching-6.png)

![](/images/basic-merging-1.png)

![](/images/basic-merging-2.png)

### 分支管理

```bash
$ git branch -v
$ git branch --merged
$ git branch --no-merged
$ git branch -d testing
```

### 远程分支

远程引用是对远程仓库的引用（指针），包括分支、标签等等。 你可以通过 `git ls-remote (remote)` 来显式地获得远程引用的完整列表，或者通过 `git remote show (remote)` 获得远程分支的更多信息。 然而，一个更常见的做法是利用远程跟踪分支。

```bash
$ git ls-remote
From https://github.com/nqiu/Blogs
5fbdce962018df477c5f0cf636abe38291b5412e        HEAD
5fbdce962018df477c5f0cf636abe38291b5412e        refs/heads/master
$ git remote show origin
* remote origin
  Fetch URL: https://github.com/nqiu/Blogs
  Push  URL: https://github.com/nqiu/Blogs
  HEAD branch: master
  Remote branch:
    master tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (fast-forwardable)
```

远程跟踪分支，以 `(remote)/(branch)` 形式命名 是远程分支状态的引用。 它们是你不能移动的本地引用，当你做任何网络通信操作时，它们会自动移动。 远程跟踪分支像是你上次连接到远程仓库时，那些分支所处状态的书签。

**拉取**

当 `git fetch` 命令从服务器上抓取本地没有的数据时，它并不会修改工作目录中的内容。 它只会获取数据然后让你自己合并。 然而，有一个命令叫作 `git pull` 在大多数情况下它的含义是一个 `git fetch` 紧接着一个 `git merge` 命令。 

```bash
$ git fetch
$ git pull
```

**推送**

当你想要公开分享一个分支时，需要将其推送到有写入权限的远程仓库上。 本地的分支并不会自动与远程仓库同步——你必须显式地推送想要分享的分支。 这样，你就可以把不愿意分享的内容放到私人分支上，而将需要和别人协作的内容推送到公开分支。

```bash
$ git push origin master
# 删除远程分支
$ git push origin --delete testing
```

### 分支变基

在 Git 中整合来自不同分支的修改主要有两种方法：`merge` 以及 `rebase`。

![](/images/basic-rebase-1.png)

```bash
$ git checkout master
$ git merge experiment
```



![](/images/basic-rebase-2.png)

其实，还有一种方法：你可以提取在 `C4` 中引入的补丁和修改，然后在 `C3` 的基础上应用一次。 在 Git 中，这种操作就叫做 *变基*。 你可以使用 `rebase` 命令将提交到某一分支上的所有修改都移至另一分支上，就好像“重新播放”一样。

![](/images/basic-rebase-3.png)

```bash
$ git checkout experiment
$ git rebase master
```



![](/images/basic-rebase-4.png)

这两种整合方法的最终结果没有任何区别，但是变基使得提交历史更加整洁。 你在查看一个经过变基的分支的历史记录时会发现，尽管实际的开发工作是并行的，但它们看上去就像是串行的一样，提交历史是一条直线没有分叉。一般我们这样做的目的是为了确保在向远程分支推送时能保持提交历史的整洁。

**变基的风险**: 不要对在你的仓库外有副本的分支执行变基。