---
title: "css in js"
date: 2019-12-10T23:48:44+08:00
lastmod: 2019-12-10T23:48:44+08:00
draft: false
keywords: ["daily"]
description: "css in js"
author: "youting"
tags: ["css"]
---

难不成是因为天下大势，分久必合，合久必分？

传统前端推崇的“关注点分离”原则，推荐将 HTML、CSS、JavaScript 分离，各司其职。但是随着 React“组件化”思想的推进，已经将 HTML 用 JSX 的形式与 JavaScript 文件整合在一起，CSS 也可以通过以下写法同样的放入一个文件：

```js
const Component = () => (
  <div
    style={{
      color: "white",
      fontSize: "12px",
    }}
    onClick={handleClick}
  >
    text
  </div>
);
```

但是这样写不支持复杂的嵌套、选择器等特性，使用起来不太方便。这里介绍几个 css-in-js 的库和一些基本用法！

## [styled-component](https://www.styled-components.com/)

> 基本用法： [codeSandbox](https://codesandbox.io/s/priceless-firefly-qqf33)

```js
import React from "react";
import Styled from "styled-components";

const Button = Styled.a`
  border-radius: 3px
  // ...
`;

function App() {
  return <Button>Learn React</Button>;
}
```

模板字符串也可以用 object 代替：

```js
const Button = Styled.a({
  borderRadius: "3px",
  // ...
});
```

使用模板字符串或者传入 object 来生成一个组件，直接使用便可。

> 全局样式：

```js
import { createGlobalStyle } from "styled-components";
const GlobalStyle = createGlobalStyle`
  div {
    margin: 20px
  }
`;

return (
  <>
    <GlobalStyle />
    <div>
      <h3> initial button </h3>
    </div>
  </>
);
```

## [emotion](https://emotion.sh/docs/introduction)

> 基本用法： [codeSandbox](https://codesandbox.io/s/emotion-stw5q)

```js
/** @jsx jsx */
import { jsx } from "@emotion/core";

const baseBtn = {
  borderRadius: "3px",
  // ...
};

return <button css={baseBtn}>Learn React</button>;
```

emotion 支持给 css 属性传入 object 格式的样式数据，但是想生效需要遵循以下两种方法其一：

- 可以自定义 babelrc 文件的，引入一个 babel preset：

```js
// .babelrc
{
  "presets": ["@emotion/babel-preset-css-prop"]
}
```

- 不可以自定义 babelrc 的，（如使用了`create-react-app`）可以引入一个注释：

```js
/** @jsx jsx */ // 对，就是这个
import { jsx } from "@emotion/core";
```

两种途径都是一个目的，用 emotion 的`jsx`方法来解析 jsx 而不是用`React.createElement`。

css 属性也可以用模板字符串：

```js
/** @jsx jsx */
import { jsx, css } from "@emotion/core";

const baseBtnWithTag = css`
  border-radius: 3px;
  // ...
`;

return <button css={baseBtnWithTag}>Learn React</button>;
```

还可以用类似 `styled-components` 的方法（模板字符串或者 object 都可以）：

```js
import styled from "@emotion/styled";

const Button = styled.a`
  border-radius: 3px;
  // ...
`;

return <Button>Learn React</Button>;
```

> 全局样式：

```js
import { jsx, css, Global } from "@emotion/core";
const global = {
  div: {
    margin: "20px",
  },
};

return (
  <>
    <Global styles={css(global)} />
    <div>
      <h3> initial button </h3>
    </div>
  </>
);

// 或者是

return (
  <>
    <Global
      styles={css`
        div {
          margin: 20px;
        }
      `}
    />
    <div>
      <h3> initial button </h3>
    </div>
  </>
);
```

> 为什么同样的样式规则，展示出来的却不一样呢！

自我感觉是因为`1em`是相对根元素的尺寸，用 css 属性注入的样式保留了浏览器给 button 的原生样式，但是生成新 Button 的方法没有保留，所以他们相对的根元素是不一样的，样式就不一样啦！

## [jss](https://cssinjs.org/?v=v10.0.0)

> 基本用法：[codeSandbox](https://codesandbox.io/s/react-jss-ejljf)

```js
import jss from "jss";
import preset from "jss-preset-default";

jss.setup(preset());

const styles = {
  button: {
    borderRadius: "3px",
    // ...
  },
};

function App() {
  const { classes } = jss.createStyleSheet(styles).attach();
  return <button className={classes.button} />;
}
```

> 全局样式：

```js
const styles = {
  "@global": {
    div: {
      margin: "20px",
    },
  },
};
```

## 别的资料

- [awesome css in js](https://github.com/tuchk4/awesome-css-in-js)
- [choosing a css in js library](https://gist.github.com/troch/c27c6a8cc47b76755d848c6d1204fdaf)
