---
title: "How to Change Commit Message"
date: 2019-10-10T16:04:23+08:00
draft: false
---

> 在使用 git 的过程中经常有修改上一个 commit message 的需求。这个命令老是爱忘记，特地记录一下。

```

# 修改上一个 commit message, 使用 --amend 进入 vim 编辑模式

$ git commit --amend

# 修改上3个 commit message, 使用 --amend 进入 vim 编辑模式

$ git rebase -i HEAD~3
```