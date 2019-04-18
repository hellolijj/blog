---
title: "hugo 发布第一篇博客	"
date: 2019-04-08T09:40:03+08:00
draft: false

---


> 按照网上大多数的博客，可以很快使用 hugo 创建一个个人博客静态网站。甚至支持自定义选择皮肤、上传至github，使用github的pages功能构建个人博客网站。但是当需要创建内容的时候，网上的中文参考资料并不是很多。本博客旨在深入挖掘 hugo 功能，最后使用 hugo 完成日常常用功能——发博文，分类，打标签。

## content 文件夹下的内容

参考文档 [pages bundles](https://gohugo.io/content-management/organization/#page-bundles)

对 contents 文件夹下的内容如下

```
.
└── content
    └── about
    |   └── _index.md  // <- https://example.com/about/
    ├── post
    |   ├── _index.md   // <- https://example.com/post/
    |   ├── firstpost.md   // <- https://example.com/post/firstpost/
    |   ├── happy
    |   |   └── ness.md  // <- https://example.com/post/happy/ness/
    |   └── secondpost.md  // <- https://example.com/post/secondpost/
    └── quote
        ├── first.md       // <- https://example.com/quote/first/
        └── second.md      // <- https://example.com/quote/second/
```

在安装hugo完成后，默认情况下，只能使用访问 post/firstpost.md 或者 post/happy/ness.md 形式访问。 

故而将本篇博客放在 post/hugo 目录下 通过 http://hellolijj.github.io/post/hugo/start-first-blog/ 路径访问。

> 理解部分: hugo 将每一个访问页面称为 Bundle 

### hugo 路径分解

```
                    section
                    ⊢--^--⊣
                               url
                    ⊢-------------^------------⊣

      baseURL             path        slug
⊢--------^--------⊣ ⊢------^-----⊣⊢----^------⊣
                  permalink
⊢----------------------^-----------------------⊣
https://example.com/events/chicago/lollapalooza/
```
