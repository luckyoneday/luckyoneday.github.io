---
title: "E2E测试中常报的错误汇总"
date: 2020-06-10T10:47:22+08:00
lastmod: 2020-06-10T10:47:22+08:00
draft: false
keywords: ["daily"]
description: "E2E测试中常报的错误汇总"
author: "youting"
tags: ["Xpath", "e2e"]
---

希望可以做成一个汇总贴，保持更新啊~

## 超时

```bash
Timeout - Async callback was not invoked within the 20000ms timeout specified by jest.setTimeout.Timeout
```

这个原因是超时啦！比如在等倒计时或者是等验证码的时候，超出了 timeout 时间就会报这个错，还会伴随着这样的报错信息：

```bash
Teardown Puppeteer
(node:73761) UnhandledPromiseRejectionWarning: Error: Caught error after test environment was torn down

Protocol error (Runtime.callFunctionOn): Target closed.
```

就是测试还没跑完环境已经关闭辽~这个时候有多种方法解决，比如直接设置：

```js
jest.setTimeout(50000);
```

或者是在单个测试的时候设置超时时间：

```js
test("xxx", () => {
  // expect(...)
}, 50000);
```

## 元素下标

Xpath 中获取元素是从 1 开头的，不像数组是 0 开头。

## Xpath 错误

```bash
Evaluation failed: TypeError: Failed to execute 'evaluate' on 'Document': The result is not a node set, and therefore cannot be converted to the desired type.
```

一般这样的报错是 Xpath 错误。

## 如何验证 Xpath 的正确性

可以在 chrome 控制台中输入 `$x('//*[text()="xxx"]')` 来获取元素，鼠标放在元素上元素会高亮来验证 Xpath 的正确性与否。

## 参考资料

- [超时 - stackOverflow](https://stackoverflow.com/questions/49603939/async-callback-was-not-invoked-within-the-5000ms-timeout-specified-by-jest-setti)
