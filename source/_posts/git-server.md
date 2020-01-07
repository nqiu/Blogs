---
title: git服务器
date: 2020-01-05 14:04:08
tags:
---

为了使用 Git 协作功能，需要一个共享的git服务器，即远程的 Git 仓库。一个远程仓库通常只是一个裸仓库（bare repository）——即一个没有当前工作目录的仓库。 因为该仓库仅仅作为合作媒介，不需要从磁盘检查快照；存放的只有 Git 的资料。 简单的说，裸仓库就是你工程目录内的 `.git` 子目录内容，不包含其他资料。架设一台git服务器并不难，选择服务器使用的通讯协议，了解它们的优缺点及适用场景，做好这些协议的典型配置即可。

## 协议

Git 可以使用四种主要的协议来传输资料：本地协议（Local），HTTP 协议，SSH（Secure Shell）协议及 Git 协议。 

### 本地协议

最基本的就是 *本地协议（Local protocol）* ，其中的远程版本库就是硬盘内的另一个目录。 这常见于团队每一个成员都对一个共享的文件系统（例如一个挂载的 NFS）拥有访问权，或者比较少见的多人共用同一台电脑的情况。 

```bash
$ git clone /opt/git/project.git
$ git clone file:///opt/git/project.git
```

如果在 URL 开头明确的指定 `file://`，那么 Git 的行为会略有不同。 如果仅是指定路径，Git 会尝试使用硬链接（hard link）或直接复制所需要的文件。 如果指定 `file://`，Git 会触发平时用于网路传输资料的进程，那通常是传输效率较低的方法。指定 `file://` 的主要目的是取得一个没有外部参考（extraneous references）或对象（object）的干净版本库副本– 通常是在从其他版本控制系统导入后或一些类似情况需要这么做。

### HTTP协议

Git 通过 HTTP 通信有两种模式: *智能（Smart） HTTP 协议* 和 *哑（Dumb） HTTP 协议*。

“智能” HTTP 协议的运行方式和 SSH 及 Git 协议类似，只是运行在标准的 HTTP/S 端口上并且可以使用各种 HTTP 验证机制，这意味着使用起来会比 SSH 协议简单的多，比如可以使用 HTTP 协议的用户名／密码的基础授权，免去设置 SSH 公钥。

智能 HTTP 协议或许已经是最流行的使用 Git 的方式了，它即支持像 `git://` 协议一样设置匿名服务，也可以像 SSH 协议一样提供传输时的授权和加密。 而且只用一个 URL 就可以都做到，省去了为不同的需求设置不同的 URL。 如果你要推送到一个需要授权的服务器上（一般来讲都需要），服务器会提示你输入用户名和密码。 从服务器获取数据时也一样。

如果服务器没有提供智能 HTTP 协议的服务，Git 客户端会尝试使用更简单的“哑” HTTP 协议。 哑 HTTP 协议里 web 服务器仅把裸版本库当作普通文件来对待，提供文件服务。 哑 HTTP 协议设置起来简单。 基本上，只需要把一个裸版本库放在 HTTP 根目录，设置一个叫做 `post-update` 的挂钩就可以了。

```bash
$ git clone https://example.com/gitproject.git
```

### SSH协议

架设 Git 服务器时常用 SSH 协议作为传输协议。 因为大多数环境下服务器已经支持通过 SSH 访问 —— 即使没有也很容易架设。 

```bash
$ git clone ssh://user@server/project.git
$ git clone user@server:project.git
```

### Git协议

Git 协议是包含在 Git 里的一个特殊的守护进程；它监听在一个特定的端口（9418），类似于 SSH 服务，但是访问无需任何授权。 要让版本库支持 Git 协议，需要先创建一个 `git-daemon-export-ok` 文件 —— 它是 Git 协议守护进程为这个版本库提供服务的必要条件 —— 但是除此之外没有任何安全措施。 要么谁都可以克隆这个版本库，要么谁也不能。 这意味着，通常不能通过 Git 协议推送。 由于没有授权机制，一旦你开放推送操作，意味着网络上知道这个项目 URL 的人都可以向项目推送数据。

## 搭建git服务器

在开始架设 Git 服务器前，需要把现有仓库导出为裸仓库——即一个不包含当前工作目录的仓库。

```bash
$ git clone --bare my_project my_project.git
```

整体上效果相当如下命令

```bash
$ cp -Rf my_project/.git my_project.git
```

把裸仓库放到服务器上

```bash
$ scp -r my_project.git user@git.example.com:/opt/git
```

此时，其他通过 SSH 连接这台服务器并对 `/opt/git` 目录拥有可读权限的使用者，通过运行以下命令就可以克隆你的仓库。

```console
$ git clone user@git.example.com:/opt/git/my_project.git
```

如果一个用户，通过使用 SSH 连接到一个服务器，并且其对 `/opt/git/my_project.git` 目录拥有可写权限，那么他将自动拥有推送权限。

如果到该项目目录中运行 `git init` 命令，并加上 `--shared` 选项，那么 Git 会自动修改该仓库目录的组权限为可写。

```console
$ ssh user@git.example.com
$ cd /opt/git/my_project.git
$ git init --bare --shared
```

如果你有一台所有开发者都可以用 SSH 连接的服务器，架设一个仓库就这么简单。

如果需要团队里的每个人都对仓库有写权限，又不能给每个人在服务器上建立账户，那么提供 SSH 连接就是唯一的选择了。 我们假设用来共享仓库的服务器已经安装了 SSH 服务，而且你通过它访问服务器。

有几个方法可以使你给团队每个成员提供访问权。 第一个就是给团队里的每个人创建账号，这种方法很直接但也很麻烦。第二个办法是在主机上建立一个 *git* 账户，让每个需要写权限的人发送一个 SSH 公钥，然后将其加入 git 账户的 `~/.ssh/authorized_keys` 文件。 这样一来，所有人都将通过 *git* 账户访问主机。 这一点也不会影响提交的数据——访问主机用的身份不会影响提交对象的提交者信息。另一个办法是让 SSH 服务器通过某个 LDAP 服务，或者其他已经设定好的集中授权机制，来进行授权。 只要每个用户可以获得主机的 shell 访问权限，任何 SSH 授权机制你都可视为是有效的。

## 配置ssh公钥

本机上生成ssh密钥对

```bash
$ ssh-keygen -t rsa 
```

一路回车，默认存储在~/.ssh/目录中，假设生成的密钥对为~/.ssh/id_rsa.john.pub和~/.ssh/id_rsa.john。

服务器上创建git账户，并为其创建.git目录

```bash
$ sudo adduser git
$ su git
$ cd
$ mkdir .ssh && chmod 0700 .ssh
$ touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

将开发者本地的公钥加入服务器git账户的authorized_keys文件中

```bash
# 本地拷贝到服务器
$ scp ~/.ssh/id_rsa.john.pub git@server:/tmp/id_rsa.john.pub
# 服务器上
$ cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys

# 或者在本地机直接运行
$ ssh-copy-id git@server
```

目前所有（获得授权的）开发者用户都能以系统用户 `git` 的身份登录服务器从而获得一个普通 shell。 如果你想对此加以限制，则需要修改 `passwd` 文件中（`git` 用户所对应）的 shell 值。

借助一个名为 `git-shell` 的受限 shell 工具，你可以方便地将用户 `git` 的活动限制在与 Git 相关的范围内。

```bash
$ sudo chsh -s /usr/bin/git-shell git 
```

## git守护进程

Git 协议都是相对容易设定的。 通常，你只需要以守护进程的形式运行该命令：

```console
git daemon --reuseaddr --base-path=/opt/git/ /opt/git/
```

`--reuseaddr` 允许服务器在无需等待旧连接超时的情况下重启，`--base-path` 选项允许用户在未完全指定路径的条件下克隆项目，结尾的路径将告诉 Git 守护进程从何处寻找仓库来导出。 如果有防火墙正在运行，你需要开放端口 9418 的通信权限。

## Smart HTTP

一般通过 SSH 进行授权访问，通过 git:// 进行无授权访问，但是还有一种协议可以同时实现以上两种方式的访问。 设置 Smart HTTP 一般只需要在服务器上启用一个 Git 自带的名为 `git-http-backend` 的 CGI 脚本。 设置 Smart HTTP 一般只需要在服务器上启用一个 Git 自带的名为 `git-http-backend` 的 CGI 脚本。 该 CGI 脚本将会读取由 `git fetch` 或 `git push` 命令向 HTTP URL 发送的请求路径和头部信息，来判断该客户端是否支持 HTTP 通信，如果 CGI 发现该客户端支持智能（Smart）模式，它将会以智能模式与它进行通信，否则它将会回落到哑（Dumb）模式下。

## GitWeb与第三方托管GitLab

如果你对项目有读写权限或只读权限，你可能需要建立起一个基于网页的简易查看器。 Git 提供了一个叫做 GitWeb 的 CGI 脚本来做这项工作。GitLab 是一个数据库支持的 web 应用，所以相比于其他 git 服务器，它的安装过程涉及到更多的东西。

**用户和组管理**

GitLab 上的用户指的是对应协作者的帐号。 用户帐号没有很多复杂的地方，主要是包含登录数据的用户信息集合。 每一个用户账号都有一个 **命名空间** ，即该用户项目的逻辑集合。 如果一个叫 `jane` 的用户拥有一个名称是 `project` 的项目，那么这个项目的 url 会是 http://server/jane/project 。

移除一个用户有两种方法。 “屏蔽（Blocking）” 一个用户阻止他登录 GitLab 实例，但是该用户命名空间下的所有数据仍然会被保存，并且仍可以通过该用户提交对应的登录邮箱链接回他的个人信息页。

而另一方面，“销毁（Destroying）” 一个用户，会彻底的将他从数据库和文件系统中移除。 他命名空间下的所有项目和数据都会被删除，拥有的任何组也会被移除。 这显然是一个更永久且更具破坏力的行为，所以很少用到这种方法。

一个 GitLab 的组是一些项目的集合，连同关于多少用户可以访问这些项目的数据。 每一个组都有一个项目命名空间（与用户一样），所以如果一个叫 `training` 的组拥有一个名称是 `materials` 的项目，那么这个项目的 url 会是 http://server/training/materials 。每一个组都有许多用户与之关联，每一个用户对组中的项目以及组本身的权限都有级别区分。 权限的范围从 “访客”（仅能提问题和讨论） 到 “拥有者”（完全控制组、成员和项目）。

**项目管理**

一个 GitLab 的项目相当于 git 的版本库。 每一个项目都属于一个用户或者一个组的单个命名空间。 如果这个项目属于一个用户，那么这个拥有者对所有可以获取这个项目的人拥有直接管理权；如果这个项目属于一个组，那么该组中用户级别的权限也会起作用。

每一个项目都有一个可视级别，控制着谁可以看到这个项目页面和仓库。 如果一个项目是 *私有* 的，这个项目的拥有者必须明确授权从而使特定的用户可以访问。 一个 *内部* 的项目可以被所有登录的人看到，而一个 *公开* 的项目则是对所有人可见的。 注意，这种控制既包括 git “fetch” 的使用也包括对项目 web 用户界面的访问。

**钩子**

GitLab 在项目和系统级别上都支持钩子程序。 对任意级别，当有相关事件发生时，GitLab 的服务器会执行一个包含描述性 JSON 数据的 HTTP 请求。 这是自动化连接你的 git 版本库和 GitLab 实例到其他的开发工具，比如 CI 服务器，聊天室，或者部署工具的一个极好方法

### 参考

* [使用Smart HTTP和Gitweb搭建简易个人git服务器](https://blog.csdn.net/IceTeaSet/article/details/59157213)