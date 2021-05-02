---
title: "Math"
date: 2020-01-10T11:46:17+08:00
lastmod: 2020-01-10T11:46:17+08:00
draft: false
keywords: ["daily"]
description: ""
author: "youting"
mathjax: true
tags: ["JavaScript"]
---

## [`Math.hypot()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/hypot)

返回它的所有参数的平方和的平方根。

```js
Math.hypot(9); // 9 等同于 Math.hypot(-9) 等同于Math.abs(x)
Math.hypot(3, 4); // 5 等同于 Math.sqrt(3*3 + 4*4)
Math.hypot(); // 0 不传入参数则返回 0
Math.hypot(NaN); // NaN
Math.hypot(3, "foo"); // NaN
Math.hypot(3, "4"); // 5 会转换为数字
```

## [`Math.sign()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/sign)

返回一个数字的符号，指示数字是正数、负数还是 0。共有 5 种返回值, 分别是 1, -1, 0, -0, NaN. 代表的各是正数, 负数, 正零, 负零, NaN。

| 返回值   | 1    | -1   | 0    | -0   | NaN |
| -------- | ---- | ---- | ---- | ---- | --- |
| 对应类型 | 正数 | 负数 | 正零 | 负零 | NaN |

```js
Math.sign(3); //  1
Math.sign(-3); // -1
Math.sign("-3"); // -1 会转换为数字
Math.sign(0); //  0
Math.sign(-0); // -0
Math.sign(NaN); // NaN
Math.sign("foo"); // NaN
Math.sign(); // NaN
```
