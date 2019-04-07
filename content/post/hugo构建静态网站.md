---
title: "Hugo构建静态网站"
date: 2018-10-09T19:00:25+08:00
---
# 3.4.1 hugo构建静态网站

@(3.4 hugo)[blog, 个人网站]


## 一些资料汇总

hugo官方网站：https://gohugo.io
hugo官方中文文档：http://www.gohugo.org/doc/
hugo github项目：https://github.com/gohugoio/hugo
hugo 安装参考：[利用Hugo和Github Pages免费创建并永久托管网站](https://cloud.tencent.com/developer/article/1326514)
hugo 主题安装： [从Hexo迁移到Hugo-送漂亮的Hugo Theme主题](http://www.flysnow.org/2018/07/29/from-hexo-to-hugo.html)


> hugo 适合于构建个人网站。因为对于大多数程序猿来说，都有使用markdown的习惯。而hugo 可以很方便将自己的markdown文档生成静态网站，上传至github。 或者gitee，即可以方便的，无成本的构建个人网站。
> 1、Hugo is a fast and modern static site generator written in Go, and designed to make website creation fun again.

## 开始使用

### 安装hugo

```
brew install hugo
hugo version
```

### 创建站点

```
hugo new site hellolijj

```

### 下载站点主题

```
cd hellolijj
git init
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
```
在配置文件config.toml里加一行，配置本网站的主题：

```
theme = "xhugo"
```

### 新建一个页面

```
hugo new posts/hello.md
```

### 预览页面

```
hugo server
```
然后打开 localhost:1313就可以浏览页面

> 注意，打开页面会发现显示出来的框架但是显示不出来内容。 github上有一个issue:[hugo server 以后博客内容为空白](https://github.com/rujews/maupassant-hugo/issues/7)


>  在下载theme的时候 git clone 与 git submodule add 的区别。一个项目（文件夹）只能有1个git clone 其他的是 git submodule add 

todo: 关于hugo如何撰写文章以后有时间再探索


## hugo项目的github部署

安装教程操作，发现将源代码上传至 `hello.github.io` 后不能选择 文件夹访问 只能通过 `https://hellolijj.github.io/docs/` 访问，显示这不是我们想要的。
因此需要设置两个项目，一个用于存放hugo源文件，另一个存放hugo 编译后生成的文件。


blog双项目不知道怎么处理，下次遇到了再说啊吧

