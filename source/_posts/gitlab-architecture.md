---
title: gitlab架构
date: 2020-01-02 16:30:49
tags:
---

## 引言

gitlab基于相同的核心，有两个版本：社区版(Gitlab-CE, Community Edition)和企业版(Gitlab-EE, Enterprise Edition)

社区版是开放源码的，公开源码在[这里]("https://gitlab.com/gitlab-org/gitlab-foss/" "gitlab-foss")，拥有MIT许可证；企业版是在社区版的基础上增加了其它的功能和特性，在专有许可证下使用，代码在[这里]("https://gitlab.com/gitlab-org/gitlab/" "gitlab")。

两种版本都需要gitlab-shell和gitaly两种必须的附加组件。这两种组件分别在[Gitlab-Shell]("https://gitlab.com/gitlab-org/gitlab-shell")和[gitaly]("https://gitlab.com/gitlab-org/gitaly")代码库。

## 架构

先通过一张简化的构架图来理解Gitlab的架构及工作原理。

![Gitlab简化架构图](/images/gitlab_architecture_simplified.png)

先介绍一下各组件：

* Nginx: Web入口，也可使用Apache，HTTP/HTTPS方式访问仓库
* Gitlab-Shell: 相对HTTP方式，使用ssh访问仓库
* PostgreSQL: 持久化数据库，也可使用MySQL，存储用户、权限、issue、MR等信息
* Sidekiq: 消费redis中的任务，如发送电子邮件、通知等
* Redis: 非持久化存储，用作缓存、任务队列等
* Unicorn: gitlab的web服务器，提供gitlab api等，处理快速的/一般的任务，与redis配合，主要包括：
  * 检查Redis中的会话进行认证、鉴权
  * 生成Sidekiq任务并发送到Redis
  * 提供项目、仓库等信息
* Gitaly: 以RPC方式提供仓库服务，专门负责访问磁盘以高效处理git操作，并缓存耗时任务。几乎所有仓库服务最终都需要gitaly。
* Gitlab-Workhorse: 反向代理，缓解Unicorn压力，处理于Rails无关的操作，管理websocket长链接，处理"Large"HTTP请求，如文件上传/下载，Git push/pull及archive下载等。



从架构图上可知，Gitlab通过web和ssh两种方式提供仓库服务。下面简要介绍一下两种方式。

* web方式
  * Nginx或Apache作为入口，接收用户请求，passproxy到gitlab-workhorse，gitlab-workhorse访问unicorn提供的gitlab api进行鉴权。对于静态资源如`$GIT_HOME/gitlab/public`下的静态页、头像、附件及其它预编译的静态资源可跳过Unicorn，由Nginx或gitlab-workhorse直接访问磁盘。
  * 可以认为所有web请求都由gitlab-workhorse接管，有些请求要交给Unicorn处理，有些gitlab-workhorse直接处理了（可能需要gitlab-workhorse先通过Unicorn鉴权后处理）。
  * Gitlab的web服务是ruby rails框架实现的，Unicorn作用Web服务器，使用PostgreSQL作为持久化数据库，使用Sidekiq处理异步任务，Redis作为缓存和任务队列。

* ssh方式
  Gitlab-Shell组件使得gitlab可以通过ssh提供仓库服务。它使用`$GIT_HOME/.ssh/authorized_keys`存储和管理ssh keys，该文件不应手动修改。Gitlab-shell通过gitaly提供的rpc操作/访问仓库，并可能提交任务到redis以供Sidekiq处理

###### 注1：$GIT_HOME为git用户的家目录，即`GIT_HOME=~git`。也可通过`cd ~git && pwd`

###### 注2：上图中的连线（进程间通信），除特别标明是TCP方式通信，默认使用socket通信

最后附一张Gitlab全组件的详细架构图，以供参考。

*todo*

## 布局与运维

采用Omnibus Package方式安装的gitalb默认安装在git的家目录，

* /var/opt/gitlab, git家目录，可通过如下命令查看

  ```bash
  $ cat /etc/passwd | grep ^git
  ```

* 目录结构如下

  ```bash
  $ tree -a -L 1 /var/opt/gitlab
  /var/opt/gitlab
  ├── alertmanager/
  ├── backups/
  ├── bootstrapped
  ├── gitaly/
  ├── git-data/
  ├── gitlab-ci
  │   └── builds/
  ├── gitlab-exporter
  │   ├── gitlab-exporter.yml
  │   └── RUBY_VERSION
  ├── gitlab-rails
  │   ├── etc/
  │   ├── REVISION
  │   ├── RUBY_VERSION
  │   ├── shared/
  │   ├── sockets/
  │   ├── tmp/
  │   ├── upgrade-status/
  │   ├── uploads/
  │   ├── VERSION
  │   └── working/
  ├── gitlab-shell/
  ├── gitlab-workhorse/
  ├── grafana/
  ├── logrotate/
  ├── nginx/
  ├── node-exporter
  │   └── textfile_collector
  ├── postgres-exporter/
  ├── postgresql
  │   └── data/
  ├── prometheus/
  ├── public_attributes.json
  ├── redis/
  ├── .ssh/
  └── trusted-certs-directory-hash
  ```

  * Gitlab在/var/opt/gitlab/bin下了rake任务，用于数据备份、查看版本、检查配置文件等
  * 各模块都有一个独立目录，其中包含了配置文件或日志文件
  * Git仓库存储在git-data目录

* Gitlab日志位置/var/opt/gitlab/log/目录，其他模块在各自的子目录或有独立的日志，如

  * redis可能在/var/log/redis/
  * nginx可能在/var/log/nginx/

* gitlab相关进程可通过如下命令查看

  ```bash
  $ ps aux | grep ^git
  ```

* gitlab提供了gitlab-ctl命令管理gitlab服务，如查看相关服务运行状态

  ```bash
  $ sudo gitlab-ctl status
  ```

  `gitlab-ctl`详细使用见参考[1]

## 参考

* [GitLab官方文档]("https://docs.gitlab.com/ee/development/architecture.html#gitlab-workhorse" "gitlab architecture")
* [gitlab官方代码库]("https://gitlab.com/gitlab-org" "gitlab.org")
* [gitlab-ctl手册]("https://docs.gitlab.com/omnibus/maintenance/" "gitlab-ctl")

