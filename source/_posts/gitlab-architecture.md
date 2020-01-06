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

  `gitlab-ctl`详细使用见参考

### gitlab服务构成

```bash
$ sudo gitlab-ctl service-list
alertmanager*
gitaly*
gitlab-exporter*
gitlab-workhorse*
grafana*
logrotate*
nginx*
node-exporter*
postgres-exporter*
postgresql*
prometheus*
redis*
redis-exporter*
sidekiq*
unicorn*
```

### gitlab邮件配置

```bash
# file /etc/gitlab/gitlab.rb
gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "xxx"
gitlab_rails['smtp_password'] = "xxx"
gitlab_rails['smtp_domain'] = "smtp.qq.com"
gitlab_rails['smtp_authentication'] = 'plain'
gitlab_rails['smtp_enable_starttls_auto'] = true
```

使配置生效

```bash
$ sudo gitlab-ctl reconfigure
# 清除缓存
$ sudo  RAILS_ENV=production gitlab-rake cache:clear
```

### gitlab配置HTTPS

创建目录用于存放ssl证书

```bash
$ sudo mkdir /etc/gitlab/ssl
$ sudo chmod 0700 /etc/gitlab/ssl
```

上传证书，并修改证书权限

```bash
$ sudo chmod 0600 /etc/gitlab/ssh/*
```

修改gitlab配置

```bash
# file /etc/gitlab/gitlab.rb
external_url "https://gitlab.xxx.com"
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.xxx.com.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.xxx.com.key"
```

使配置生效

```bash
$ sudo gitlab-ctl reconfigure
```

在防火墙上开放443端口，用于HTTPS

```bash
$ sudo iptables -I INPUT -m tcp -p tcp --dport 443 -j ACCEPT
```

### 修改root用户密码

打开rails程序，交互控制台

```bash
$ sudo gitlab-rails console production
```

修改root密码

```bash
# gitlab-rails console production
Loading production environment (Rails 4.2.8)
irb(main):001:0> user = User.where(id: 1).first
=> #<User id: 1, email: "admin@example.com"......
irb(main):009:0> user.password = '12345678'
=> "12345678"
irb(main):010:0> user.password_confirmation = '12345678'
=> "12345678"
irb(main):011:0> user.save!
Enqueued ActionMailer::DeliveryJob (Job ID: 510bb5be-a156-4522-9983-44d8a895e92a) to Sidekiq(mailers) with arguments: "DeviseMailer", "password_change", "deliver_now", gid://gitlab/User/1
=> true
irb(main):011:0> exit
```

### 运维管理

```bash
# 查看版本
cat /opt/gitlab/embedded/service/gitlab-rails/VERSION

# 检查gitlab
gitlab-rake gitlab:check SANITIZE=true --trace

# 实时查看日志
gitlab-ctl tail

# 数据库关系升级
gitlab-rake db:migrate

# 清理redis缓存
gitlab-rake cache:clear

# 升级GitLab-ce 版本
yum update gitlab-ce

# 升级PostgreSQL最新版本
gitlab-ctl pg-upgrade
```

### 服务管理

```bash
# 启动所有 gitlab 组件：
gitlab-ctl start

# 停止所有 gitlab 组件：
gitlab-ctl stop

# 停止所有 gitlab postgresql 组件：
gitlab-ctl stop postgresql

# 停止相关数据连接服务
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq

# 重启所有 gitlab 组件：
gitlab-ctl restart

# 重启所有 gitlab gitlab-workhorse 组件：
gitlab-ctl restart  gitlab-workhorse

# 查看服务状态
gitlab-ctl status

# 生成配置并启动服务
gitlab-ctl reconfigure
```

### 日志

```bash
# 实时查看所有日志
gitlab-ctl tail

# 实时检查redis的日志
gitlab-ctl tail redis

# 实时检查postgresql的日志
gitlab-ctl tail postgresql

# 检查gitlab-workhorse的日志
gitlab-ctl tail gitlab-workhorse

# 检查logrotate的日志
gitlab-ctl tail logrotate

# 检查nginx的日志
gitlab-ctl tail nginx

# 检查sidekiq的日志
gitlab-ctl tail sidekiq

# 检查unicorn的日志
gitlab-ctl tail unicorn
```

### 数据备份

GitLab备份的默认目录是 /var/opt/gitlab/backups ，如果想改备份目录，可修改/etc/gitlab/gitlab.rb

```bash
# file /etc/gitlab/gitlab.rb
gitlab_rails['backup_path'] = '/data/backups'
# 备份保留天数，单位：s
gitlab_rails['backup_keep_time'] = 604800
```

使配置生效

```bash
$ sudo gitlab-ctl reconfigure
```

执行备份，可使用crontab进行自动备份

```bash
$ sudo gitlab-rake gitlab:backup:create
```

### 数据恢复

假如使用如下的备份恢复数据

```console
/var/opt/gitlab/backups/1499244722_2020_01_05_12.6.1_gitlab_backup.tar
```

停止 unicorn 和 sidekiq ，保证数据库没有新的连接，不会有写数据情况。

```bash
# 停止相关数据连接服务
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq

# 指定恢复文件，会自动去备份目录找。确保备份目录中有这个文件。
# 指定文件名的格式类似：1499244722_2020_01_05_12.6.1，程序会自动在文件名后补上：“_gitlab_backup.tar”
# 一定按这样的格式指定，否则会出现 The backup file does not exist! 的错误
gitlab-rake gitlab:backup:restore BACKUP=1499244722_2020_01_05_12.6.1

# 启动Gitlab
gitlab-ctl start
```

## 参考

* [GitLab官方文档]("https://docs.gitlab.com/ee/development/architecture.html#gitlab-workhorse" "gitlab architecture")
* [gitlab官方代码库]("https://gitlab.com/gitlab-org" "gitlab.org")
* [gitlab-ctl手册]("https://docs.gitlab.com/omnibus/maintenance/" "gitlab-ctl")

