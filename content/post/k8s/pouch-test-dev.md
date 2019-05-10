---
title: "Pouch Test Dev"
date: 2019-05-10T15:52:28+08:00
draft: false
---

> 对于一个开源项目来说，了解它的测试流程，搭建测试环境是对理解代码有一定好处的。本文参照 pouch 测试文档，搭建测试环境，记录此过程中的一点思考。

## pouch 测试开发

> 按照 pouch 的文档。pouch 的测试环境要基于 linux 环境（centos 或者 ubutu）, mac 上搭建此环境是不行的。原因在于 pouch 还不支持 mac 版本。（mac 的 docker实现方式与linux不一样）。 但是在 mac 上搭建 moby 项目的开发环境还是可以运行的。

参考官方的测试文档：http://pouchcontainer.io/#/pouch/docs/test/test.md 一步步搭建没有遇到什么问题。单元测试，通过 `make unit-test` 就可以完成单元测试。单个文件单个函数的测试可以借助 ide 完成。

```
# pouchd -D --enable-lxcfs=true --lxcfs=/usr/bin/lxcfs >/tmp/log 2>&1 &
# pouch pull registry.hub.docker.com/library/busybox:latest 
```

pouchd 的启动过程中。需要依赖 containerd、runc、lxcfs，安装的脚本在 hack/install 目录下。 使用的lxcfs前要进行文件挂载 `lxcfs /var/lib/lxcfs`

## 如何进行集成测试

除了单元测试外，也需要进行集成测试。集成测试的方法如下：

```
cd test
# 测试某个文件
go test -check.f PouchHelpSuite
# 测试某个文件下的某个方法
go test -check.f PouchHelpSuite.TestHelpWorks
```
这里参数来自于每个测试文件的 struct 结构