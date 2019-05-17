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
pouch api 的文档是由 `apis/swagger.yml` 文件维护。pouch 项目中的 `apis/types` 由此文档自动生成。还有docs/api/HTTP_API.md 也由这个文档自动生成
> moby 项目还可以根据 swagger.yml 产生 html 网址文档，也启动 pouch 能早日实现。

2、



