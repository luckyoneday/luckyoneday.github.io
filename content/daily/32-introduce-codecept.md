---
title: 介绍 codeceptjs
date: 2020-08-07T17:27:18+08:00
lastmod: 2020-08-07T17:27:18+08:00
draft: false
toc: true
keywords: ["daily"]
description: ""
tags: ["e2e", "BDD"]
author: "youting"
---

## 为什么选用 [codeceptjs](https://codecept.io/basics/#getting-started)

- BDD 测试： https://codecept.io/bdd/#what-is-behavior-driven-development
- 可以和 cucumber 结合： https://www.youtube.com/watch?v=mIasShmyw9I
- 可以选各种测试 helper： https://codecept.io/basics/#architecture
- 可以测试 mobile： https://codecept.io/mobile/#mobile-testing-with-appium
- 可以跨浏览器： https://help.crossbrowsertesting.com/selenium-testing/frameworks/codeceptjs/
- 还有各种插件： https://codecept.io/plugins/#pauseonfail
- 方便的定位 locator： https://codecept.io/locators/#locators

## 怎么写

[上篇](/daily/30-introduce-cucumber)里面有提到 cucumber 的步骤定义可以用自己喜欢的任意框架来写。codeceptjs 支持集成 cucumber，并且提供了方便的命令来根据 feature 描述生成 steps definition 文件框架，就可以使用 codeceptjs 的 API 来写测试脚本从而验证 feature 文件的步骤描述。

codeceptjs 的入口是一个叫做 codecept.conf.js 的文件，里面会有各种配置包括使用的 helpers 是 puppeteer 还是 webDriver，plugin 有哪些等。

如果要集成 gherkin，则在配置文件中增加这样一项：

```js
gherkin: {
  features: '*.feature',
  // 自定义步骤可以不需要手动进入到配置文件中
  steps: './steps_definition/steps.js'
}
```

就会识别出 .feature 结尾的文件然后运行时去匹配 steps 配置项下的文件中的描述，继而运行相应的测试用例。

codeceptjs 还提供了`codeceptjs gherkin:snippets [--path=PATH] [--feature=PATH]`命令来根据 feature 文件中的描述生成如下的代码框架：

```js
Given("在xx页面", () => {
  // From "feature/xxx.feature" {"line":4,"column":6}
  throw new Error("Not implemented yet");
});

When("点击xx按钮", () => {
  // From "feature/xxx.feature" {"line":5,"column":7}
  throw new Error("Not implemented yet");
});

Then("打开xx弹窗", () => {
  // From "feature/xxx.feature" {"line":6,"column":5}
  throw new Error("Not implemented yet");
});
```

然后就可以使用 codeceptjs 提供的 API 来具体实现相应的步骤：

```js
Given("在xx页面", () => {
  I.amOnPage("/index");
});

When("点击xx按钮", () => {
  I.click("#button");
});

Then("打开xx弹窗", () => {
  I.see("xx header");
});
```

最后使用 `codeceptjs run --features` 来运行。

## 几个注意事项

- codeceptjs 配置 puppeteer 时，会发现启动的 chromium 尺寸在修改 `windowSize` 后也没有什么变化，此时需要额外的一些配置：

```js
helpers: {
      // ...
      Puppeteer: {
        windowSize: '1280x1000',
        chrome: { args: ['--window-size=1280,1000'], defaultViewport: null }
        // ...
      },
}

```

- 当遇到跨域情况时，可以直接在配置文件中写：

```js
helpers: {
      // ...
      Puppeteer: {
        chrome: { args: ['--disable-web-security'] }
        // ...
      },
}

```

- codeceptjs 可以在自定义 helper 里面拿到 Puppeteer 的 `page` 和 `browser`

```js
// config.js
helpers: {
      // ...
      CustomerHelper: {
        require: "path/to/CustomHelper";
      }
}

// CustomHelper.js
const Helper = codecept_helper

class CustomHelper extends Helper {
      // ...
      const { browser } = this.helpers.Puppeteer
      const { page } = this.helpers.Puppeteer
}
```
