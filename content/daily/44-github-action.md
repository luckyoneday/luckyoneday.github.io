---
title: "github action"
date: 2021-05-03T16:32:34+08:00
lastmod: 2021-05-03T16:32:34+08:00
draft: false
toc: false
keywords: ["daily"]
description: "博客发布方式修改--github-action"
tags: ["git"]
author: "youting"
---

之前博客发布是使用本地运行 `deploy.sh` 的方式，脚本可见 [deploy.sh](../4-deploy)，这种方式每次发布都重新生成 `public` 文件夹，需要重新进行 git 初始化操作，导致远端 github 仓库永远只有一次 commit，看不到变更操作。所以决定修改为由 github action 在 push 代码时触发 hugo 来构建 github page。

触发 github action 需要在项目根目录下新建 `.github/workflows` 文件夹，并新建一个 `.yml` 结尾的文件，内容参考 [actions-hugo](https://github.com/peaceiris/actions-hugo)

```yml
name: CI
on:
  push:
    branches:
      - master
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v2.3.4
        with:
          submodules: true
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2.4.13
        with:
          hugo-version: "0.80.0"
          extended: true
      - name: Build
        run: hugo
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3.8.0
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          external_repository: username/username.github.io
          publish_branch: master
          publish_dir: ./public
```

## deploy_key

关于 deploy_key 可见 [action-gh-pages](https://github.com/peaceiris/actions-gh-pages)。

## sub modules

项目根目录下需要有一个 `.gitmodules` 文件，内容为

```
[submodule "themes/even"]
  path = themes/even
  url = https://github.com/olOwOlo/hugo-theme-even.git
[submodule "public"]
  path = public
  url = https://github.com/username/username.github.io.git
  branch = master
```

{{% admonition info tips %}}

之前使用 even 主题的时候，本地有对源代码进行修改，但是如果直接链接作者的源 git 仓库会发现自己的修改是没有用的。于是 fork 了一份，并将 `.gitmodules` 中的 `themes/even` submodule 的 url 修改为自己 fork 后仓库的 url，发现可以生效~

{{% /admonition %}}

## 参考

- [HUGO + Github + Github Action 持续集成部署个人博客](https://blog.csdn.net/weixin_41263449/article/details/107584336)
- [actions-hugo](https://github.com/peaceiris/actions-hugo)
- [action-gh-pages](https://github.com/peaceiris/actions-gh-pages)
