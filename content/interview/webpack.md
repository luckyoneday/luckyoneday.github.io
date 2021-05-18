---
title: "webpack"
date: 2021-05-04T11:02:08+08:00
lastmod: 2021-05-04T11:02:08+08:00
draft: true
toc: true
keywords: ["interview"]
description: "webpack 面试知识点"
tags: ["interview"]
author: "youting"
---

## 用处

- 实现前端项目的模块化
- 使用一些高级的特性来加快开发效率或者安全性，例如通过 ES6+、TypeScript 开发脚本逻辑，通过 sass、less 等方式来编写 css 样式代码
- 监听文件的变化并且反映到浏览器上，提高开发效率
- HTML、CSS 也需要模块化
- 代码压缩、合并等一些优化

## loader

loader 的配置写在 module.rules 属性中，属性介绍如下：

- `rules` 是一个数组的形式，所以支持配置很多个 `loader`
- 每一个 `loader` 对应一个对象的形式，对象属性 `test` 为匹配的规则，一般情况为正则表达式
- 属性 `use` 针对匹配到文件类型，调用对应的 `loader` 进行处理

执行方向从右向左。

```js
// ...
module: {
  rules: [
    {
      test: /\.css$/,
      use: [
        { loader: "style-loader" },
        {
          loader: "css-loader",
          options: {
            modules: true,
          },
        },
        { loader: "sass-loader" },
      ],
    },
  ];
}
```

常见的 loader：

- style-loader: 将 css 添加到 DOM 的内联样式标签 style 里
- css-loader: 允许将 css 文件通过 require 的方式引入，并返回 css 代码
- less-loader: 处理 less
- sass-loader: 处理 sass
- file-loader: 分发文件到 output 目录并返回相对路径
- url-loader: 和 file-loader 类似，但是当文件小于设定的 limit 时可以返回一个 Data Url
- babel-loader :用 babel 来转换 ES6 文件到 ES5

## plugin

运行在 webpack 打包的各个阶段。贯穿了整个 webpack 编译周期。

本质是一个具有 `apply` 属性的 JavaScript 对象。

- mini-css-extract-plugin: 提取 CSS 到一个单独的文件中
- HtmlWebpackPlugin: 打包结束后，⾃动生成⼀个 html ⽂文件，并把打包生成的 js 模块引⼊到该 html 中

## loader 和 plugin 的区别

- loader 运行在打包文件之前，处理各种文件
- plugin 在整个编译周期都起作用

## Tree Shaking

依赖 ES Module 的静态语法分析。在 webpack 实现 Tree Shaking 有两种不同的方案：

- usedExports：通过标记某些函数是否被使用，之后通过 Terser 来进行优化
- sideEffects：跳过整个模块/文件，直接查看该文件是否有副作用
