---
title: "Gsoc Think List"
date: 2019-06-26T11:22:01+08:00
draft: true
---

很幸运参加了 2019 年的 GSoC， 这是我第一次做类似的事情，特地写系列博客记录这两个月的学习成果。

## 项目名称

Kubernetes with hardware devices topology awareness at node level

在 kubernestes 上 基于 gpu 亲和性（拓扑）调度

## 项目背景

目前， kubernetes 发行版对 gpu 的支持力度较粗。存在以下不足

### 每一块GPU同时最多只能被一个容器使用

这也就意味着 一块gpu 只能被一个任务独占使用。 当块 gpu 显存非常大（例如 30g），而实际任务用不了那么多，这就造成了资源的浪费。

### 没有考虑GPU卡之间的通道亲和性

当一个任务使用多块gpu时，例如 1个任务需要3块 gpu, 而节点中有 8块gpu。 这个时候 kubernetes 在 8块gpu 随机选择 3块。

实际上，GPU卡直接的连接通常是不一样的，有的通过Nvlink相连，有的通过PCIe，而不同的连接方式性能差别非常大。
而没有考虑同个主机里GPU卡直接的通道亲和性时，也会给多卡计算时需要发生数据传输时(如all_reduce操作)带来过高的通信开销。

## 项目工作量

- 弄清楚 gpu 共享 的原理

- 开发实现 基于 gpu 亲和性（拓扑）的 调度

## 项目输出博客

- how to write gsoc proposal
- kuberentes 支持 gpu 原理
- device plugin in 源码解析
- gpushare 原理
- gpu 底层结构
- scheduler-extender 解析
- kuberntes 选举策略