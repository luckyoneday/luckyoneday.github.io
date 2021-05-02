---
title: "服务端渲染路由同构"
date: 2019-12-06T14:20:21+08:00
lastmod: 2019-12-06T14:20:21+08:00
draft: false
toc: false
keywords: ["daily"]
description: "服务端渲染路由同构"
author: "youting"
tags: ["SSR", "Node"]
---

由于服务端都是无状态的，所以服务端渲染与浏览器端是不一样的，基本就是用无状态的`<StaticRouter>`代替浏览器端的`<BrowserRouter>`：

```js
// client
<BrowserRouter>
  <App/>
</BrowserRouter>

// server
<StaticRoute location={ctx.url} context={{}}>
  <App/>
</StaticRouter>
```

App 组件中写 routes：

```js
// import routes from './routes'
<Switch>
  {routes.map((item) => (
    <Route path={item.path} component={item.component} key={item.path}></Route>
  ))}
</Switch>
```

但是现在只是解决了路由匹配的问题，但是如果路由不匹配想要 redirect 呢，作为静态路由的 StaticRouter 是接收不到重定向的（亲测写<Redirect />就是完全的浏览器渲染啦，会有很明显的页面闪烁）所以需要额外的处理。

在 redirect 发生的时候，打印`StaticRouter`的`context`发现多出了一些东西：

```js
{
  css: [ /* ... */ ],
  action: 'REPLACE',
  location: { pathname: '/home', search: '', hash: '', state: undefined },
  url: '/home'
}
```

然后就可以利用这个 action 和 url 在服务端也做重定向！

```js
if (context.action === "REPLACE" && context.url) {
  ctx.response.redirect(context.url);
}
```
