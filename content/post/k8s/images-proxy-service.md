---
title: "Images Proxy Service"
date: 2019-10-14T17:38:21+08:00
draft: false
---

# 镜像代理服务

## 问题

很多情况下，都会遇到一些 gcr.io 、 k8s.gcr.io 镜像无法下载的问题。这是针对国内用户的。

## 解决方法

- 使用 github dockerfile 构建镜像的方式可以解决

- 设置 docker proxy 代理

- 寻找镜像代理服务

- 通过 google云 终端的 docker pull 然后再 docker push 到阿里云的镜像仓库。然后再去使用阿里云的镜像仓库。

## 镜像代理服务

Azure 中国提供了 `gcr.io` `k8s.gcr.io` 镜像代理服务

例如：期望下载 `gcr.io/google_containers/addon-resizer:1.8.4` 的镜像。

```bash
docker pull gcr.azk8s.cn/google_containers/addon-resizer:1.8.4
```

通过执行以上命令，然后 docker tag 修改镜像名称可以解决。


