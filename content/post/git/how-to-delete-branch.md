---
title: "How to Delete Branch"
date: 2019-05-21T17:40:16+08:00
draft: false
---

> 在使用 git 的过程中经常有删除分支的需求。这个命令老是爱忘记，特地记录一下。

```

# 删除远程分支

$ git push origin --delete dev-container-prune

# 删除本机的分支

$ git branch -D dev-container-prune
```