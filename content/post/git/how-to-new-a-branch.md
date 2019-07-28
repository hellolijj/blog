---
title: "How to New a Branch"
date: 2019-07-28T16:47:01+08:00
draft: true
---

创建一个分支是开发者日常使用最多的命令之一。通常我们都这样使用

```
$ git checkout -b dev
```

但是这个情况下创建的分支往往是基于当前分支而创建的新的分支。

有时候，我们希望基于 某一个分支（很多情况下是基于某个线上的分支）来创建新的分支。这个时候使用如下命令

```
git checkout -b dev origin:master
```

> origin:master 表示 origin 远程仓库的 master 分支。
