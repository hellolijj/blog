---
title: "Why Docker Was So Hot"
date: 2019-04-25T16:45:32+08:00
draft: false
---

学习原文：https://time.geekbang.org/column/article/14254

> 这篇文章介绍了docker干掉 Cloud Foundry的背景，不竟让人docker到底做了什么事情，为什么变得这么火热起来

## 2013年的主流PaaS平台——Cloud Foundry

cloud foundry 的使用

```bash
cf push 'project'
```

用户在执行这条命令时，cloud foundry将应用的可执行文件和启动脚本打进一个压缩包内，上传至cloud foundry的存储中。启动时，cloud foundry的调度器选择一个虚拟机将应用压缩包下载下来启动。

后来改进调用操作系统的Cgroups和Namespaces创建为每个应用创建一个沙盒，在沙盒中启动应用。

缺点：为每种语言、每种框架、每个版本维护一个大包，维护配置文件。这样显得杂乱、难以管理。

## docker 的火爆

正因为cloud foundry这样的弊端，docker项目将应用运行依赖的环境（操作系统）一起打包成镜像文件。使得你的本地环境与云端运行环境高度一致。

Docker 项目之所以能取得如此高的关注，在与它解决了打包和发布这一困扰运维人员多年的技术难题。

## 其他公司

Docker 公司的对手 CoreOS, CoreOS也是一个基础设施领域的创业公司。在Docker火热的过程中，它成为了docker项目的第二重要力量。

CoreOS 公司的产品

- rkt 容器

- Container Linux 操作系统

- Fleet 作业调度工具

- Systemd 进程管理工具

- Etcd 状态存储工具

Mesosphere 创业公司

Mesos 是 Berkeley（加州大学伯克利分校）主导的大数据套件之一，是大数据火热时最受欢迎，跟 Yarn 项目是竞争产品。

Mesos 发布的 Marathon 项目通过了万台节点严重。 Mesos + Marathon 的组合实际上进化成了高度成熟的 PaaS 项目

## Docker 项目火热之后

docker 项目火热之后触及了Google、Redhat、CoreOS等公司的切身利益。容器领域其他几个玩家成立一个中立的基金会。并且提出 OCI 的规范和标准。

- Docker 公司捐出Libcontainer，并命名为RunC

Google、Redhat 等公司共同牵头发起了一个名为 CNCF 的基金会，旨在以 Kubernetes 为基础，建立一个开源基础设施领域厂商主导、按照基金会运营方式的平台级社区来对抗docker。

由于Kubernetes出色的容器编排特性，开发式的运营方式及api，是的k8s很快的火热起来。

之后

- Docker将运行时Containerd捐献给CNCF社区，docker开源项目改为为Moby。

- 2017年 Docker 宣布 docker 企业版中内置kubernetes

- 2018年 3月28日 docker 公司的cto 宣布辞职。至此纷纷扰扰的容器圈子，尘埃落定。
> todo 
1. Mesos 的两层调度机制？
调度器是Mesos的核心部件，主要负责将各个slave上资源分配给各个framework。Mesos为了支持多framework接入，采用了双层调度机制，首先，由mesos中的 allocator 将资源分配给framework，然后又由framework自己的调度器将资源分配给任务。



