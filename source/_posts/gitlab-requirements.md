---
title: gitlab requirements
date: 2020-01-08 14:59:30
tags:
---

gitlab性能影响因素很多，包括但不限于硬件、服务配置、活跃用户数、仓库数、仓库大小、自动化任务种类、频率等等。

## gitlab硬件要求

### 存储

两个因素: 容量和IOPS。

根据gitlab上托管项目数量、每个项目占用磁盘大小、未来使用情况进行规划，当然可使用空间越大越好。考虑到便于水平扩展，建议使用LVM方式管理磁盘分区。

如果有足够的内存和CPU处理性能，Gitlab的整体性能将受限于磁盘的导道时间，使用高转速的磁盘(7200转以上）或SSD可以大大提高Gitlab响应时间。

### CPU

1核CPU，最大支持100用户，所有worker及后台任务都落在一个核上，导致Giblab响应缓慢；

2核CPU，是官方推荐的支持100用户的最低要求；

4核CPU，最大支持500用户；

8核CPU，最大支持1000用户；

32核，最大支持5000用户；

更多用户，请参考[Gitlab官方HA方案](https://about.gitlab.com/solutions/high-availability/)

### Mem

至少需要8GB的可寻址空间(RAM+Swap)，操作系统和其它的应用程序也会消耗内存；至少4GB的可用空间留给Gitlab使用；内存不足，Gitlab使用过程中会出现500等错误。

4GB RAM + 4GB Swap，最大支持100用户，且响应很慢；

8G RAM，是官方推荐的支持100用户的最低要求；

16G RAM，最大支持500用户；

32G RAM，最大支持1000用户；

128G RAM，最大支持5000用户；

更多用户，请参考[Gtilab官方HA方案](https://about.gitlab.com/solutions/high-availability/)

即使有足够的RAM，也建议使用至少配置2GB的Swap，防止系统内存波动时，Gitlab出错的几率。同时建议将内核参数swappiness设置为比较低的值，如10，以便让Gitlab尽可能的使用RAM。

```bash
$ cat /proc/sys/vm/swappiness
10
```

表示，当RAM使用率达到90%左右的时候才会频繁使用swap。详参[设置swappiness参数](https://askubuntu.com/questions/103915/how-do-i-configure-swappiness/103916#103916)

## 服务配置要求

### 数据库

根据用户数、项目数进行规划，建议至少保留5GB磁盘空间给数据库，Gitlab使用PostgreSQL 9.6版本或以上。确定每个数据库使用pg_trgm扩展。在每个数据库实例运行如下命令开启

```bash
> CREATE EXTENSION pg_trgm
```

该扩展程序用于快速搜索字符串。

### Unicorn Worker

对于每个gitlab实例，建议使用 `CPU Cores x 1.5 + 1`个Unicorn Worker。比如对于4核的节点应该使用7个Unicorn Worker。

### Redis & Sidekiq

Redis用来存储所有用户的会话信息、以及后台任务队列。Redis内存要求较小，平均每个用户25KB。Sidekiq是采用多线程的进程处理后台任务，至少200MB+起，随着时间推移，对于10000活跃用户的内存会超过1GB。

## 参考

* [Gitlab官方高可用方案](https://about.gitlab.com/solutions/high-availability/)
* [多节点Omnibus高可用部署](https://docs.gitlab.com/omnibus/update/#multi-node--ha-deployment "Multi-node / HA deployment")
* [Gitlab高可用配置](https://docs.gitlab.com/ee/administration/high_availability/gitlab.html#first-gitlab-application-server)
* [Gitaly高可用配置](https://docs.gitlab.com/ee/administration/high_availability/gitaly.html#configuring-gitaly-for-scaled-and-high-availability)
* [Redis高可用配置](https://docs.gitlab.com/ee/administration/high_availability/redis.html#provide-your-own-redis-instance-core-only)
* [PostgreSQL高可用配置](https://docs.gitlab.com/ee/administration/high_availability/database.html#provide-your-own-postgresql-instance-core-only-1)