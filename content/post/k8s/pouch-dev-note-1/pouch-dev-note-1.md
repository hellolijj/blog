---
title: "Pouch Dev Note (1)"
date: 2019-05-17T20:14:50+08:00
draft: false
---

## Pouch 开发笔记

> 最近突然发现阿里巴巴推出了一个 [SOC2019](https://developer.aliyun.com/summerofcode2019) 的项目，也就是阿里巴巴编程之夏。这是他们第一次搞，这是在向谷歌编程之夏（GSOC）学习。由于对这个感兴趣，就关注了一下。并提出了一个 `issue` [pouch support search in cli](https://github.com/alibaba/pouch/issues/2839)。 随后又自己想开发这个特性，自己就开始看 pouch 源码，并动手写了。

> 这里记录一下开发过程中的一些想法。

### pouch cli

开发遇到的第一个问题，就是怎么添加 search 命令的入门。这就要求了解 pouch cli 的写法。pouch 也引用了 cobra 的写法。

1、入口文件

pouch cli 的入口在 `cli/cli.go` 文件里。包括setflag操作，设置日志操作都是在这个文件里。这个文件 
```
rootCmd: &cobra.Command{
	Use:   "pouch",
	Short: "An efficient container engine",
	Long:  pouchDescription,
	// disable displaying auto generation tag in cli docs
	DisableAutoGenTag: true,
}
```
定义了一级命令、和对于这个命令的描述。

2、二级命令文件

这里的二级命令是我自己定义的。例如 `pouch info` 这里的 info 就是二级命令。例如，我希望增加一个 `pouch search` 的命令。那么我就创建一个 `cli/search.go`的文件。

Init 函数里有关于这个命令的用法。
RunE 是执行这个命令要运行的函数。
Args 是对参数的要求。

```
func (s *SearchCommand) Init(c *Cli) {
	s.cli = c

	s.cmd = &cobra.Command{
		Use:   "search [OPTIONS] TERM",
		Short: "Search the images from specific registry",
		Long:  searchDescription,
		Args:  cobra.MinimumNArgs(1),
		RunE: func(cmd *cobra.Command, args []string) error {
			return s.runSearch(args)
		},
		Example: searchExample(),
	}
	s.addFlags()
}
```

例如: 执行 `pouch search nginx` 命令, 就会执行 runSearch() 函数。其中 addFlags() 函数是对 `pouch search nginx -r www.a.com` 里的 -r 参数进行解析。

3、命令的具体实现——构造client端请求

一般在 cli 目录下只是简单的对参数解析判断，调用 APIClient 的接口。真正在 cli 端做事情的是实现这些接口的函数。一般位于 client 目录下。

本特性是调用 ImageSearch 接口。具体的实现在 `client/image_search.go` 文件里。

因为pouch项目是采用c/s架构。一条命令一般由客户端发送一个请求。daemon 负责相应。因此 client 目录下的大多都是构造请求参数，然后发送请求的 daemom。 pouch search 构造请求代码如下：
```
var results []types.SearchResultItem

q := url.Values{}
q.Set("term", term)
q.Set("registry", registry)

headers := map[string][]string{}
if encodedAuth != "" {
	headers["X-Registry-Auth"] = []string{encodedAuth}
}

resp, err := client.post(ctx, "/images/search", q, nil, headers)

if err != nil {
	return nil, err
}

err = json.NewDecoder(resp.Body).Decode(&results)
return results, err
```

### Pouch API

pouch api 是 pouch-cli 与 pouch-daemon 之间通信的接口。 pouch api 代码位于 apis 目录下。

1、api 文档维护。
pouch api 的文档是由 `apis/swagger.yml` 文件维护。pouch 项目中的 `apis/types` 由此文档自动生成。还有docs/api/HTTP_API.md 也由这个文档自动生成。

> moby 项目还可以根据 swagger.yml 产生 html 网址文档，也启动 pouch 能早日实现。

2、api 路由

前文 [what happen pouchd](./what-happen-pouchd/what-happen-pouchd)我已经分析过，执行 `pouchd` 后，最后会启动一个 server 监听 客户端的请求。监听服务器文件在 `apis/server/server.go` 

server 收到请求后，会分发给对应的 handler。 这一部分处理的逻辑位于 `apis/server/router.go`文件。

### Pouch Daemon 

在 router.go 里的 handler 处理函数，最终会跳转到 xxx_bridge.go 文件。例如：image search 就跳转到 `apis/server/image_bridge.go#searchImages` 文件。

bridge 文件一般都是接受参数，对参数判断。然后调用 xxxMgr 接口。例如 image search 就是调用 ImageMgr 接口的 SearchImages 方法。

xxxMgr 的实现位于 `daemon/mgr` 路径下。 这个文件夹里有所有对 pouch 对象的处理方法。例如 对于 pull image 操作。对应的处理文件是 `daemon/mgr/image.go#SearchImages`。 

对于某些比较复杂的操作。比喻 PullImage 操作。还会调用 mgr.client.FetchImage 方法。
mgr.client 实际上 containerd 的client。

### Containerd Client

pouch 对于底层容器、镜像的操作实际上是依赖于 containerd 的。安装 pouch 实际上也安装了（必须安装containd）而 containerd 由 依赖于 runc。（runV只有涉及到才要求安装）。

pouch 对于 contaninerd 的操作之前创建一个 ctrd 的对象（这个步骤在 pouchd 启动时候创建）。ctrd 的实现位于 `ctrd/` 目录下。

### pouch 命令流程

简单总结以上过程可绘制以下流程图

![pouch-flow-progress](../pouch_flow_progress.png)

## pouch 测试

对于一个开源项目，完整的测试体系是十分有必要的。在pouch项目中有 单元测试 和集成测试。相关测试方法可以参照 我之前写的文章 [pouch test dev](../../pouch-test-dev)

## pouch github

上传github的过程有很多地方需要注意。

1、git commit 需要签名

由于 github 项目需要统计你的信息。因此 每次上传都要 `git commit -s -m "some info"`

2、code check

不同的项目有不同的风格。go 代码风格不是有以下几点

（1）、每次上传都要进行 golint

这就要求你的开发环境需要有 golint。golint 是代码质量检查工具。它可以将代码中不规范的地方指引出来。

> 问题：golint 可以检测单个文件 或者某目录下的所有文件。它无法递归检测。 

（2） 每次上传都要进行 gofmt

gofmt 操作有时候

gofmt –w program.go gofmt结构覆盖原来文件

（3）每次上传都要进行 goimport

import 包要求：第一部分使用系统包、第二部分使用本项目包、第三部分使用第三方包。两个部分之间使用空行隔开，不同包之间安装字母顺序排序。

（4） todo 部分使用大写 TODO

（5）使用日志返回错误信息的时候，注意大小写形式。
https://github.com/golang/go/wiki/CodeReviewComments#error-strings






