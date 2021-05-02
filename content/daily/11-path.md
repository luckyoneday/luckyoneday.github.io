---
title: "记录下各种 path"
date: 2019-12-09T08:43:03+08:00
lastmod: 2019-12-09T08:43:03+08:00
draft: false
toc: false
keywords: ["daily"]
description: "记录下各种 path"
author: "youting"
tags: ["path"]
---

// 本来是想写服务端渲染热更新 -- server 端，emmmmm 没配置好，换个话题 :joy:

总是看别人的代码里面有这样的写法`path.resolve(__dirname, 'src')`，自己去写的时候却并没有按照理所当然认为的拼接路径来...so 记录一下

> [path.resolve](https://nodejs.org/docs/latest/api/path.html#path_path_resolve_paths) :arrow_down_small:

`path.resolve([...paths])`中的`...paths`是一系列路径字符串，从右到左处理，后面的一个 item 会被拼在前面，直到拼出**绝对路径**（差不多看到`/`就是绝对路径了，就不拼了）；如果拼到最后面也没有拼成绝对路径，就会用当前工作目录路径。

```js
path.resolve("/foo", "/bar", "baz"); // /bar/baz
path.resolve("/foo/bar", "./baz"); // /foo/bar/baz
path.resolve("/fool/bar", "/tmp/file"); ///tmp/file
path.resolve("wwwroot", "static_files/png/", "../gif/image.gif");
// 如果当前目录是/Usr/username/node, 那么会返回/Usr/username/node/wwwroot/static_files/gif/image.gif
```

> [path.join](https://nodejs.org/docs/latest/api/path.html#path_path_join_paths) :arrow_down_small:

`path.join([...paths])`中的`...paths`也是一系列路径字符串，使用平台特定的分隔符作为分隔符将所有给定的路径参数连接在一起，然后对结果路径进行规范化。

```js
path.join("/foo", "/bar", "baz"); // /foo/bar/baz
path.join("/foo/bar", "./baz"); // /foo/bar/baz
path.join("/fool/bar", "/tmp/file"); // /fool/bar/tmp/file
path.join("wwwroot", "static_files/png/", "../gif/image.gif");
// wwwroot/static_files/gif/image.gif

path.join("a", "b1", "..", "b2"); // a/b2
path.join("a", "b1", "b2", ".."); // a/b1
```

> [output](https://webpack.js.org/concepts/output/) :arrow_down_small:

`output`配置项告诉 webpack 打包后的文件存放的路径和文件名。

```js
output: {
  filename: 'bundle.js',
  path: path.resolve(__dirname, 'dist')
}
```

> [publicPath](https://webpack.js.org/guides/public-path/) :arrow_down_small:

`publicPath`允许我们指定程序的静态资源的地址。在实际开发过程中，我们可能会把静态资源放在与首页同级的`assets/`文件夹下，但是一旦我们想在生产环境把静态资源上传 CDN 呢？

`publicPath`常用于生产环境，它会为资源指定一个基础的路径，比如上面的`output`增加了`publicPath`的配置之后：

```js
output: {
  filename: 'bundle.js',
  path: path.resolve(__dirname, 'dist'),
  publicPath: '/public/'
}
```

本地 webpack-dev-server 启动会提示：

```sh
ℹ ｢wds｣: Project is running at http://localhost:8080/
ℹ ｢wds｣: webpack output is served from /public/
ℹ ｢wds｣: Content not from webpack is served from ./dist
```

就只能通过 `localhost:8080/public/` 才能访问到首页。

若是使用了 html-webpack-plugin 设置的 webpack-dev-server 的 publicPath 最好与 output 的 publicPath 保持一致，（因为创建模板引用资源是基于 output 的 path ）否则会报找不到资源的错误。

```js
devServer: {
  contentBase: "./dist";
  publicPath: "/public/";
}
```

> 其他的一些 :arrow_down_small:

- [process.cwd](https://nodejs.org/api/process.html#process_process_cwd): 返回当前 Node.js 进程工作目录路径。

- [\_\_dirname](https://nodejs.org/docs/latest/api/modules.html#modules_dirname): 当前被执行文件的文件夹所在绝对路径。

- [\_\_filename](https://nodejs.org/docs/latest/api/modules.html#modules_filename): 当前执行文件所在绝对路径。
