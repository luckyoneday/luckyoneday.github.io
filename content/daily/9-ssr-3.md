---
title: "服务端渲染添加样式"
date: 2019-12-07T15:29:24+08:00
lastmod: 2019-12-07T15:29:24+08:00
draft: false
toc: false
keywords: ["daily"]
description: "关于服务端渲染的样式"
author: "youting"
tags: ["SSR", "Node"]
---

客户端打包用[`mini-css-extract-plugin`](https://github.com/webpack-contrib/mini-css-extract-plugin)将 antd.css 抽出来作为单独的 css 文件，在 HTML 文件中引入；将 \*.module.scss 用[`ignore-loader`](https://github.com/cherrry/ignore-loader) ignore 掉不处理（因为已经在服务端初始化时被打入 HTML 文档）。

服务器端使用[`isomorphic-style-loader`](https://github.com/kriasoft/isomorphic-style-loader)通过 React Router 提供的 context 对象将 \*.module.scss 文件生成的样式打入文档的`<style>`标签；使用[`ignore-loader`](https://github.com/cherrry/ignore-loader)忽略 css 文件（因为会作为静态资源引入文档，不需要打入 HTML 了）。

> 为什么要用`isomorphic-style-loader`呢！

因为这个 loader 可以将类名和样式注入 HTML 文档，作用同[`style-loader`](https://github.com/webpack-contrib/style-loader)，但是`style-loader`中用了 window 对象，在 node 端不能使用。那为什么需要注入文档的`<style>`标签，而不单独搞出来 css 文件像 antd.css 一样外部引用呢，因为用的 css modules，需要用这个插件注入局部作用域的标识符，如果解析成 css 文件那么类名就不匹配了。

> 那`isomorphic-style-loader`是怎么个工作流程呢！

这个插件提供了两个帮助方法`_insetCss()`（将 css 注入 DOM）和`_getCss()`（返回样式字符串）。可以利用最外层的 Router 的 context 对象传递样式。思路就是在渲染组件的时候，在组件内接收 context 对象，获取组件样式，放到 context 中，服务端拿到样式，插入到返回的 HTML 中的 style 标签中。

```js
export default function WithStylesHoc(Component, styles) {
  return class NewComponent extends React.Component {
    constructor(props) {
      super(props);
      if (this.props.staticContext) {
        const css = styles._getCss();
        this.props.staticContext.css.push(css);
      }
    }
    render() {
      return <Component {...this.props} />;
    }
  };
}
```

上面的高阶组件可以在 `constructor` 的时候拿到样式文件并传递给 props 中的 staticContext，

那样式是怎么传入的呢！这就要在 server.js 里：

```js
const context = { css: [] };
const content = ReactDOMServer.renderToString(
  <StaticRouter context={context} location={ctx.request.url}>
    <App staticContext={context} />
  </StaticRouter>
);

// render 完后 context.css 上已经挂了东西
const cssStr = context.css.length ? context.css.join("\n") : "";
ctx.body = template.replace("/* this will be replaced by css */", cssStr);
```

使用：

```js
import * as React from "react";
import WithStylesHoc from "./withStylesHOC";
import Styles from "./index.module.scss";

function HomePage(props: any) {
  return (
    <section className={Styles.formSection}>
      <div>这是主页</div>
    </section>
  );
}
export default WithStylesHoc(HomePage, Styles);
```
