---
title: "关于服务端渲染"
date: 2019-12-04T10:51:12+08:00
lastmod: 2019-12-04T10:51:12+08:00
draft: false
toc: false
keywords: ["daily"]
description: "React服务端渲染的一些问题"
author: "youting"
tags: ["SSR", "Node"]
---

> 本地打包完的 server.bundle.js 运行时，浏览器拿到的 js 文件返回也是个 HTML

很绝望，还报了`Unexpected token <`的错误，js 文件返回的开头竟然是`<!DOCTYPE HTML>`，后来发现是因为设置的静态资源`Static("dist")`文件夹不对。

> 浏览器端和服务端的`entry`：

client 使用[`hydrate`](https://reactjs.org/docs/react-dom.html#hydrate)，用来接收通过`ReactDOMServer`渲染的 HTML，并且 React 会复用服务端传回来的 HTML ，把事件绑定到已存在的文档上。

```js
import React from "react";
import ReactDOM from "react-dom";
import App from "./src/App";

ReactDOM.hydrate(<App />, document.getElementById("root"));
```

server

```js
import { renderToString } from "react-dom/server";
import App from "./src/App";

// 产生html
const content = renderToString(<App />);
const template = fs.readFileSync(process.cwd() + "/dist/index.html", "utf-8");
const html = template.replace(
  "<!-- this will be replaced by render  -->",
  content
);
ctx.body = html;
```

> `ReactDOMServer` 的 API：

- [`renderToString`](https://reactjs.org/docs/react-dom-server.html#rendertostring): 返回 HTML 字符串，可以使用这个 API 来服务器端生成 HTML 并在首次请求时返回客户端。

- [`renderToStaticMarkup`](https://reactjs.org/docs/react-dom-server.html#rendertostaticmarkup): 与`renderToString`类似，只是这个方法不会生成 React 依赖的内部属性，如`data-reactroot`，这个方法适合生成静态页面，因为减少属性值可以节省一些字节，但是如果想生成之后会有交互的页面，不要使用这个方法。

- [`renderToNodeStream`](https://reactjs.org/docs/react-dom-server.html#rendertonodestream): 返回一个[Readable stream](https://nodejs.org/api/stream.html#stream_readable_streams)，并且输出一个 HTML 字符串，这个字符串和`renderToString`的返回就是一样的啦。区别在于这个 API 只在服务端可用，且`renderToString`是读取完所有的 HTML 之后再返回，而`renderToNodeStream`以“流”的形式塞给`response`对象，不会等所有的 HTML 都渲染出来才返回。

```js
const content = renderToNodeStream(<App />).pipe(response);
```

- [`renderToStaticNodeStream`](https://reactjs.org/docs/react-dom-server.html#rendertostaticnodestream): 这个和`renderToStaticMarkup`就很像啦，也是适合返回静态页面。
