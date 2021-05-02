---
title: "关于登录状态"
date: 2019-11-29T13:36:19+08:00
lastmod: 2019-11-29T13:36:19+08:00
draft: false
keywords: ["daily"]
description: "koa-session"
author: "youting"
tags: ["Koa", "Node"]
---

没有使用 jwt 鉴权，因为 cookie 有大小限制，就总觉得少存一些东西比较好，而且对「退出登录」怎么处理 jwt 没有搞清楚，前端可能是清除 cookie，那后端怎么鼓捣没有想通！

所以选用了 cookie 中存 sessionId，redis 存储 session 这种方式，并且 koa 有相应的封装好的中间件[koa-session](https://github.com/koajs/session)和[koa-redis](https://github.com/koajs/koa-redis)。为啥用 redis...emmm 其实也是问了下水哥，他说之前他们都是存在 redis 里，因为登录信息访问比较多而且 redis 比较快。（然后我也是傻瓜操作，`keys *`列出 sessionId，`get [key]`看下信息就没别的了）。

## 关于跨域

在使用[koa-session](https://github.com/koajs/session)时，如果不外接存储 session 的 store，会默认将 session 信息也存到 cookie 里面，[koa-redis](https://github.com/koajs/koa-redis)封装了[koa-session](https://github.com/koajs/session)外接 store 所需要的`get`、`set`和`delete`方法用以在 redis 中存取数据。

当在 login 接口操作`ctx.session = {...}`时，koa-session 会向接口返回 set-Cookie 字段，在 response header 中能看到，但是在浏览器 application 的 cookies 里没看到，后来才知道是前端木有允许跨域，axios 上挂`axios.defaults.withCredentials = true`就可以了。

后端跨域用了[koa2-cors](https://github.com/zadzbw/koa2-cors)这个中间件：

```js
import * as cors from "koa2-cors";

app.use(
  cors({
    origin: () => "http://localhost:8080",
    exposeHeaders: ["WWW-Authenticate", "Server-Authorization"],
    maxAge: 5,
    credentials: true,
    allowMethods: ["GET", "POST", "DELETE"],
    allowHeaders: ["Content-Type", "Authorization", "Accept"],
  })
);
```

## 关于 type

在对`ctx.session`进行`get`和`set`操作时，typeScript 报错`Property 'session' does not exist on type 'ParameterizedContext<any, IRouterParamContext<any, {}>>'.`

解决方法：目前的解决方法是把`@types/koa`锁在`2.0.50`版本：

```js
"devDependencies": {
  "@types/koa": "2.0.50",
  // ...
},
"resolutions": {
  "@types/koa": "2.0.50"
}
```

相关 issue：https://github.com/DefinitelyTyped/DefinitelyTyped/issues/36161
