---
title: "codeceptjs Hook"
date: 2020-08-19T16:46:37+08:00
lastmod: 2020-08-19T16:46:37+08:00
draft: false
toc: true
keywords: ["daily"]
description: "codeceptjs 的 hooks"
tags: ["e2e", "BDD"]
author: "youting"
---

背景是在接 e2e 的时候先对所有接口进行了 request header 的复写（为了绕过风控），codeceptjs 的 helper 里面可以拿到 puppeteer，就可以使用 puppeteer 的 page 来操作 request header ：

```js
class CustomHelper extends codecept_helper {
  async addRequestHeader() {
    await page.setRequestInterception(true);
    // ...
    page.on("request", (request) => {
      if (request._interceptionHandled === false) {
        request.continue({
          headers: {
            ...request.headers(),
            "special-header": "special_content",
          }, // 复写 header
        });
      }
    });
  }
}
```

然后可以使用 codeceptjs 的 hook 来内置这个方法，就不需要写测试用例的时候调用了。

```js
// 增加风控
event.dispatcher.on(event.test.before, async (test) => {
  console.log(`开始注册风控在 ${test.title} 之前`);
  const { I } = inject();
  await I.addRequestHeader();
});
```

codeceptjs 中存在一些事件处理的 [event dispatcher](https://codecept.io/hooks/#api) ，例如：

- `event.test.before(test)`： 跑整个 test 之前会异步触发，类似 helpers 里面的 `_before` 和测试用例里面的 `Before` 。
- `event.test.after(test)`： 跑整个 test 之后异步触发。
- `event.test.started(test)`： 测试开始时同步进行。

同时 codeceptjs 的 helpers 中也存在相应的 [hooks](https://codecept.io/helpers/#hooks)，例如：

- `_init`： 所有测试前触发
- `_finishTest`： 所有测试后触发
- `_before`： 一个测试步骤之前
- `_after`： 一个测试步骤之后

对应关系大致如下：

```js
event.dispatcher.on(event.all.before, () => {
  runHelpersHook("_init");
});
event.dispatcher.on(event.test.before, () => {
  runAsyncHelpersHook("_before");
});
event.dispatcher.on(event.test.after, () => {
  runAsyncHelpersHook("_after", {}, true);
});
```

具体对应关系和执行时机可以看：[github](https://github.com/codeceptjs/CodeceptJS/pull/967/files#diff-6af460d7c54e75645928e6a7cb8650bf)

---

为什么会有 `request._interceptionHandled === false` 这个判断呢？是因为会报 `AssertionError [ERR_ASSERTION]: Request is already handled!` 这个错误，意思是 request 已经被处理过了，所以需要过滤一下已经被处理过的请求。

## 参考资料

- [cucumber-before-hook](https://medium.com/@priyank.it/cucumber-before-hook-vs-background-usage-f10453e81920)
- [interception-error](https://github.com/puppeteer/puppeteer/issues/2687)
