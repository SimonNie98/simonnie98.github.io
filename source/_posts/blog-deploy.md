---
title: 博客部署方案
date: 2024-06-02 06:11:12
tags: [blog]
categories: [Others]
toc: true
comments: true
---
<!-- more -->
# 1. 主题配置
使用了[Maupassant主题](https://www.haomwei.com/technology/maupassant-hexo.html).

# 2. 部署方式
这里记录下部署的方式：
## 2.1 本地渲染+Hexo一键部署
参考[官方文档](https://hexo.io/zh-cn/docs/one-command-deployment).

## 2.2 GitHub Actions
本博客目前采用了这种方式进行部署。这种方法是将代码推送到 ``username.github.io`` 仓库之后自动运行github actions进行build和deploy，这种方法一开始碰到了一个问题：因为这个博客的主题是在 ``themes/maupassant`` 文件夹下修改配置后进行部署的，build的时候会把该文件夹识别成submodule然后因为找不到submodule的url无法拉取而失败。于是我尝试 ``git submodule add xxxx`` 配置了submodule的相关信息。这样倒是部署成功了，但是起初拉取的配置是前文提到的主题中的远程仓库，也即我自己的配置没有办法被拉取到。

这个时候想到的一个solution是自己fork一份前文提到的主题，然后把本地关于改主题的修改后的配置/资源文件push到自己的主题repo，同时把自己的remote repo配置成本博客的submodule并进行更新，然后commit并push之后，能运行成功。

另一种方式：不采用submodule的方式进行主题的加载，即把所有本主题（ ``themes/maupassant`` ）文件夹下的内容统统添加并push到 ``username.github.io`` 仓库。这样的好处是比较简单粗暴，需要把 `themes/.gitkeep` 文件删掉以能正确添加该目录，从而只用一个repo就完成了对博客的管理。坏处是如果后续该主题的maintainer有什么更新无法及时拉取。
