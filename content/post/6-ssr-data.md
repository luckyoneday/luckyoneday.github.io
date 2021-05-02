---
title: "服务端渲染合集"
date: 2021-03-23T21:30:52+08:00
lastmod: 2021-03-23T21:30:52+08:00
draft: false
toc: true
keywords: ["SSR"]
description: "服务端渲染数据预取"
tags: ["SSR", "Node"]
author: "youting"
---

数据预取在服务端渲染中是很重要的一环，具体操作就是在服务端发送请求，灌入组件后使首屏页面直出；客户端也使用服务端获取的数据来保持数据同步。

<!--more-->

前面的小笔记指路：

- [服务端渲染路由同构](../../daily/8-ssr-2/)
- [服务端渲染样式添加](../../daily/9-ssr-3/)
- [服务端渲染热更新](../../daily/10-ssr-4/)

数据预取涉及到服务端渲染的两个概念：「脱水」和「注水」。

- **数据注水**：在服务端将预取的数据注入到浏览器，使浏览器端可以访问到。
- **数据脱水**：客户端进行渲染前将数据传入对应的组件，来保持 `props` 的一致。

## 具体操作

### 第一步

将路由所需的首屏数据对应的异步方法（例如 `loadData`）挂在组件上，便于服务端获取该路由内容时请求数据：

```js
// route.js
export default [
  {
    path: "/",
    component: Home,
    exact: true,
    // 服务端加载 Home 前要执行的方法
    loadData: Home.loadData,
  },
  // ...
}
```

可以使用一个高阶组件将 `loadData` 挂载到组件上：

```js
export default function withFetchHOC(Component, options) {
  return class App extends React.Component {
    static loadData = { ...options };
    // ...
  };
}

// home.jsx
function HomePage() {
  return <div>...</div>;
}

const options = {
  list: async () => await Api.getList(),
};

export default WithFetchHoc(HomePage, options);
```

### 第二步

服务端匹配到对应的路由时调用对应路由的 `loadData` 方法，并组织好数据传递给组件：

```js
import { matchRoutes } from 'react-router-config'

router.get("*", async (ctx) => {

  // 匹配路由
  const matchedRoutes = matchRoutes(routes, ctx.url) || []
  const matchLen = matchedRoutes.length

  // 如果有嵌套路由，那么最后一项是最深的子路由
  const { route: currRoute = {}, match } =
    matchLen > 0 ? matchedRoutes[matchLen - 1] : {}

  // 获取到 loadData 下挂载的 options，然后调用
  const loadData = currRoute.loadData
  const entries = Object.entries(loadData)
  const resArr = await Promise.all(
    entries.map(async ([key, fetch]) => {
      try {
        const singleRes = await fetch()

        return {
          status: "done",
          data: singleRes
        }
      } catch (e) {
        return {
          status: "pending",
          data: {}
        }
      }
    })
  )

  const finalData = entries.reduce((prev, cur, idx) => {
    prev[cur[0]] = resArr[idx]
    return prev
  }, {})

  // 注水
  script = `<script>window.__INIT_DATA__=${JSON.stringify(finalData)}</script>`

  // 将获取的数据作为 props 传入 App
  const content = ReactDOMServer.renderToString(
    <StaticRouter context={context} location={ctx.request.url}>
      <App staticContext={context}  __initData__={finalData}/>
    </StaticRouter>
  )

  // ...
  // 替换页面的 content、script
}
```

### 第三步

浏览器端可以通过 `window.__INIT_DATA__` 获取到同步数据。

```js
// 脱水
const renderMethod = (module as any).hot ? ReactDOM.render : ReactDOM.hydrate
renderMethod(
  <BrowserRouter>
    <App
      staticContext={null}
      __onedayInitData__={window.__INIT_DATA__}
    />
  </BrowserRouter>,
  root
)
```

还可以将服务端请求失败的情况进行兜底，重新请求一次。这些逻辑可以收拢在 `withFetchHOC` 中：

```js
export default function withFetchHOC(Component, options) {
  return class App extends React.Component {
    static loadData = { ...options };

    // 作为初始化数据放到 state 中
    state = { data: transToInitData(this.props.__initData__) ?? {} };

    reFetchData = async () => {
      const initData = this.props.__initData__ ?? {};
      delete window.__INIT_DATA__;

      const entries = Object.entries(options);

      try {
        const resultList = await Promise.all(
          entries.map(async ([key, fetch]) => {
            // 已经获取到的数据就直接 resolve，否则重新请求
            if (initData[key]?.status === "done") {
              return Promise.resolve(initData[key].data);
            } else {
              return await fetch();
            }
          })
        );

        const entriesObject = entries.reduce((prev, cur, idx) => {
          prev[cur[0]] = resultList[idx];
          return prev;
        }, {});

        this.setState({ data: entriesObject });
      } catch (e) {
        console.error("csr fetch error", e);
      }
    };

    componentDidMount() {
      // 页面挂载时重新获取一次数据
      this.reFetchData();
    }

    render() {
      const { __initData__, ...otherProps } = this.props;
      return <Component {...otherProps} initData={this.state.data} />;
    }
  };
}
```

### 小 tips

- 使用了 `react-router-config` 中的 `matchRoutes` 方法，可以处理嵌套路由，并且可以匹配到形如 `/detail/:hash` 的路由，同时 `match` 可以拿到 `{hash: xxxxx}` 的内容。
- 服务端和客户端可以分别维护一份 `context` ，作为 `options` 中 `fetch` 的参数回传，使 `fetch` 方法可以获取到 `query`、`hash` 等参数，进行接口请求：

```js
// 服务端
const optContext = {
  url: ctx.originalUrl ?? ctx.url,
  query: ctx.query ?? {},
  pathname: ctx.path ?? "",
  hashParams: (match && match.params) ?? {}, // matchRoutes 方法中的 match
};

// 浏览器端
const matchedRoutes = matchRoutes(routes, window.location.pathname) || [];
const matchLen = matchedRoutes.length;

const { match = undefined } = matchLen > 0 ? matchedRoutes[matchLen - 1] : {};

const optContext = {
  url: window.location.pathname + window.location.search,
  pathname: window.location.pathname,
  query: parseSearch(),
  hashParams: (match && match.params) ?? {},
};

// ...
// await fetch(optContext)

// 使用
const options = {
  detail: async (context) => {
    const params = context.hashParams;
    return await Api.getDetail({ hash: params.hash });
  },
};
```

## 参考文章

- [react-router - server-rendering](https://reactrouter.com/web/guides/server-rendering)
- [实现服务器端组件渲染](https://nirodu.com/n-book/react/react_ssr.html#%E5%AE%9E%E7%8E%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%E7%BB%84%E4%BB%B6%E6%B8%B2%E6%9F%93)
