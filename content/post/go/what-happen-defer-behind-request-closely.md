---
title: "What Happen Defer Behind Request Closely"
date: 2019-05-21T18:37:42+08:00
draft: false
---

今天遇到了一个问题，前前后后调试了一个下午。特此记录一下。

源代码如下：

```goland
resp, err := client.post(ctx, "/containers/prune", q, nil, nil)
defer resp.Body.Close()
if err != nil {
	return nil, err
}
```

> 这段代码乍得一看，也没有什么问题，正常情况下（网络请求正常返回）也不会出现问题。
> 但是一旦 服务器端关闭。客户端发送post请求失败，这时候立马 resp.BodyClose() 报地址错误。

正确应该将 defer 函数放在 err 判断的后面

```goland
resp, err := client.post(ctx, "/containers/prune", q, nil, nil)
if err != nil {
	return nil, err
}
defer resp.Body.Close()
```

特此谨记！