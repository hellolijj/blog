---
title: "Go Mod Vendor"
date: 2019-10-14T16:30:40+08:00
draft: false
---

# go mode vendor

> 最近接触到一个新的项目 `hydra`, 里面是通过 go mod 来管理 vendor 包的。

## go mode vendor 版本要求

要求 go 语言版本 1.11 以上

## 使用方式

```bash
# go mod vendor
```

## 问题

对于国内用户而言，使用 go mod vendor 最最最大的坑再有不能访问一下墙外的包。例如：`k8s.io`, `golang.io` 这些域名。

## 解决方法

网上罗列了一些方法

- 在 github 上下载对应的文件，然后拷贝到 vendor 目录下。

- 使用 replace 变量，将翻墙到包替换掉。

以上方法均不是最好途径。最佳途径是设置代理

```bash
export GO111MODULE=on && export GOPROXY=https://goproxy.io && go mod vendor
```

## 其他

出了 go 社区推出的 go proxy 代码服务外，还有几个其他的代理服务也值得推荐

- 七牛云代理站点 https://goproxy.cn
- 阿里云代理站点 https://mirros.aliyun.com/goproxy