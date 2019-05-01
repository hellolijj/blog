---
title: "Pouch First Touch"
date: 2019-04-30T16:19:41+08:00
draft: false
---

> 看了亮哥的《docker源码分析》热情澎湃，但是自己远没有这个能力。自从上次被面试后，一改之前的学习表面，学习特性的态度。今天对着 [pouch](https://github.com/alibaba/pouch) 开源项目分析了，`pouch version` 命令的一整个执行流程。


## pouch 的安装编译

对于开发者来说，能够对github上master代码进行安装编译是最基本的能力。
按照官网的[安装流程](https://github.com/alibaba/pouch/blob/master/INSTALLATION.md)来操作，一切都很顺畅。

就是在 安装 go 项目时,使用 `go get github.com/alibaba/pouch` 就行。

使用时，首先启动`pouchd`开启一个 pouch 的守护进程，然后通过 pouch 发送命令，pouchd 接受命令并执行解析，返回结果即可。

[这里放一个pouch项目的架构图]

## 客户端

pouch 项目的入口在 main 函数里。

```
func main() {
	if reexec.Init() {
		return
	}

	setupFlags(rootCmd)

	if err := rootCmd.Execute(); err != nil {
		logrus.Error(err)
		os.Exit(1)
	}
}
```
main 函数做一些参数的初始化、日志文件的初始化。最后 `pouch version` 命令落实到 `cli/version.go`文件里。

cli/version.go文件，主要做一些对 `pouch version` 命令的解释说明，以及example样子，有些代码是自动生成的。也可以通过这些说明自动生成帮助文档，接口文档。

```
func (v *VersionCommand) runVersion() error {
	ctx := context.Background()
	apiClient := v.cli.Client()

	result, err := apiClient.SystemVersion(ctx)
	if err != nil {
		return fmt.Errorf("failed to get system version: %v", err)
	}

	v.cli.Print(result)
	return nil
}
```

`runVersion()` 函数主要是调用 	`SystemVersion()`函数。 `SystemVersion()`函数是 pouch 项目的接口, 接口位于 `client/interface.go`文件中，这里定义了pouch 客户端的接口规范。

接口的实现。

真正对接口的实现，最终落实到 client 文件夹的文件中。例如 `SystemVersion` 接口的实现在 `client/system_version.go` 文件的 `SystemVersion()` 函数中。

```
func (client *APIClient) SystemVersion(ctx context.Context) (*types.SystemVersion, error) {
	resp, err := client.get(ctx, "/version", nil, nil)
	if err != nil {
		return nil, err
	}

	version := &types.SystemVersion{}
	err = decodeBody(version, resp.Body)
	ensureCloseReader(resp)

	return version, err
}
```
这个命令实际上就是通过 get 方式请求 "/version" 接口。至此，客户端的发送请求就完成了。

## 服务端

服务端实际上就是 pouchd 进程。对应的入口文件（实际上也会启动文件）在 `deamon/daemon.go` 中。在启动 pouchd 的过程中也启动了一个 server。 对应的文件在 `api/server/server.go` 文件中。 那么对应的处理的handler的路由在 `api/server/router.go` 文件中。

这时候，已经客户端发送的是 `/version` 接口，对应的处理逻辑如下：

```
func initRoute(s *Server) *mux.Router {
	r := mux.NewRouter()

	handlers := []*serverTypes.HandlerSpec{
		// system
		{Method: http.MethodGet, Path: "/_ping", HandlerFunc: s.ping},
		{Method: http.MethodGet, Path: "/info", HandlerFunc: s.info},
		{Method: http.MethodGet, Path: "/version", HandlerFunc: s.version},
		...
	}

	if s.APIPlugin != nil {
		handlers = s.APIPlugin.UpdateHandler(handlers)
	}

	// register API
	for _, h := range handlers {
		if h != nil {
			r.Path(versionMatcher + h.Path).Methods(h.Method).Handler(filter(h.HandlerFunc, s))
			r.Path(h.Path).Methods(h.Method).Handler(filter(h.HandlerFunc, s))
		}
	}

	if s.Config.Debug || s.Config.EnableProfiler {
		profilerSetup(r)
	}
	return r
}
```

然后根据对应api的接口可以找到对应的handle处理程序。返回 Version 结构体的Json给客户端。

```
func (mgr *SystemManager) Version() (types.SystemVersion, error) {
	kernelVersion := unknownKernelVersion
	if kv, err := kernel.GetKernelVersion(); err != nil {
		logrus.Warnf("Could not get kernel version: %v", err)
	} else {
		kernelVersion = kv.String()
	}

	return types.SystemVersion{
		APIVersion:    version.APIVersion,
		Arch:          runtime.GOARCH,
		BuildTime:     version.BuildTime,
		GitCommit:     version.GitCommit,
		GoVersion:     runtime.Version(),
		KernelVersion: kernelVersion,
		Os:            runtime.GOOS,
		Version:       version.Version,
	}, nil
}
```