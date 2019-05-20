---
title: "Node Topology Manager"
date: 2019-05-20T15:43:14+08:00
draft: false
---

> 今年暑假我报了一个谷歌编程之夏的项目。
> 方向：https://github.com/cncf/soc#kubernetes-with-hardware-devices-topology-awareness-at-node-level
> proposal:https://docs.google.com/document/d/10jZMwC8chOJknEwBastSQ5dYsasWG2UKT9SKNe73cDU/edit
> 这里记录我查资料的过程

## 节点上的拓扑管理器

这个议题的设计文稿在：https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/topology-manager.md

主要是由 https://github.com/lmdaly/kubernetes/tree/dev/topology_manager 这个分支在开发。

它给device增加了一个
