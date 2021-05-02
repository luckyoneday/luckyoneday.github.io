---
title: "HashRouter vs BrowserRouter"
date: 2020-06-11T21:43:56+08:00
lastmod: 2020-06-11T21:43:56+08:00
draft: false
keywords: ["daily"]
description: "HashRouter vs BrowserRouter"
author: "youting"
tags: ["React", "React-Router"]
---

## HashRouter

- 当渲染一个新路由时，使用哈希 routes 更新浏览器 URL (`/#/about`)
- 哈希不会被服务端处理，服务端只会匹配到 `/`，并且忽视后面的哈希值，所以会对所有路由请求返回 `index.html`。哈希值会被 react router 使用 `window.location.hash` 处理
- 用于支持通常不支持 HTML pushState API 的旧版浏览器
- 不需要任何服务端配置就可以处理路由
- 只是修改哈希，不会触发页面刷新

那怎么进行更新呢，可以通过监听 `hashchange` 事件：

```js
window.addEventListener("hashchange", function (event) {
  console.log(event);
});
```

## BrowserRouter

- 最广泛使用的路由，且基于 [HTML pushState API](https://developer.mozilla.org/en-US/docs/Web/API/History_API) 生成的路由
- 最常规的 URL 进行路由匹配 (`/about`)，无法通过 URL 区分是服务器渲染还是客户端渲染。
- 需要服务器来匹配请求的 URL
- 使用 `pushState` 或 `replaceState` 修改路由也不会刷新页面

### 关于 History

History 的属性：

- `History.length`: 返回在会话历史中的记录条数，包含了当前会话页面。
- `History.state`: 保存了会触发 popState 事件的方法（例如 `pushState` 和 `replaceState` ）传递过来的属性对象。

方法：

- `History.back()`: 指向浏览器会话历史中的上一页，跟浏览器的回退按钮功能相同
- `History.forward()`: 指向浏览器会话历史中的下一页，跟浏览器的前进按钮功能相同
- `History.go()`: 可以跳转到浏览器会话历史中的指定的某一个记录页。 `History.go(-1)` 和 `History.back()` 效果相同
- `History.pushState()`: 将给定的数据压到浏览器会话历史栈中，会改变当前 url，但是不会伴随着页面刷新
- `History.replaceState()`: 将当前会话页面的 url 替换为指定的 url，会改变当前 url，但是也不会刷新页面

`pushState` 和 `replaceState` 方法都接收三个参数，分别是 state 对象，title 标题和目标 url

```js
console.log(history.length); // 1

window.history.pushState({ foo: "bar" }, "page 2", "bar.html");
console.log(window.history.state); // { foo: "bar"; }

console.log(history.length); // 2
window.history.replaceState({ foo: "bar" }, "page 2", "foo.html");

console.log(history.length); // 2
```

每次触发 `history.go()` 、 `history.back()` 和 `history.forward()` 都会触发一个 `popstate` 事件：

```js
window.addEventListener("popstate", function (event) {
  console.log(event);
});
```

但是 `history.pushState()` 和 `history.replaceState()` 不会触发 `popstate` 事件。

## 参考资料

- [https://learnwithparam.com/blog/different-types-of-router-in-react-router/](https://learnwithparam.com/blog/different-types-of-router-in-react-router/)
- [https://krasimirtsonev.com/blog/article/deep-dive-into-client-side-routing-navigo-pushstate-hash](https://krasimirtsonev.com/blog/article/deep-dive-into-client-side-routing-navigo-pushstate-hash)
