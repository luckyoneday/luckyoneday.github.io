---
title: "介绍 cucumber"
date: 2020-07-10T17:24:14+08:00
lastmod: 2020-07-10T17:24:14+08:00
draft: false
toc: true
keywords: ["daily"]
description: "介绍 cucumber"
tags: ["BDD", "e2e"]
author: "youting"
---

## 关于 BDD

BDD 是 [Behaviour Driven Development](https://cucumber.io/docs/bdd/) / 行为驱动开发，是一种软件开发流程。是通过自然语言定义系统行为，以功能使用者的角度，编写需求场景，且这些行为描述可以直接形成需求文档，同时也是测试标准。

所以就是有三个要点：

- 行为驱动 -- 从用户角度描述行为
- 需求场景描述 -- 自然语言描述，就可以作为需求文档和测试标准
- 具体实现 -- 具体的步骤定义和步骤实现

所以 BDD 可以降低不同角色的人对于同一个需求的理解差异，更专注于业务。

## 关于 cucumber

cucumber 是一种可以使用文本描述语言来执行自动测试用例的工具，即支持 BDD 的测试工具之一。使用的语言叫做 Gherkin 。

cucumber 读取纯文本形式编写的可执行规范，并自动解析场景对应的步骤定义，随后检验我们的应用程序是否满足那些规范所包含的内容。

基于 cucumber 的 BDD 测试流程如下：

- `.feature` 文件： 定义需求场景和步骤描述文本
- 步骤定义： 步骤描述文本的具体实现，也就是可执行的测试代码
- 运行测试： 执行测试用例
- 生成报告

### 文本描述

`.feature` 文件使用 Gherkin 语言编写，例如以下内容：

```
Feature: Is it Friday yet?
  Everybody wants to know when it's Friday

  Scenario: Sunday isn't Friday
    Given today is Sunday
    When I ask whether it's Friday yet
    Then I should be told "Nope"
```

关键字介绍：

- `Feature` : 一个文件是从 `Feature` 开始的，是一个功能的名称。`Feature` 到 Background 之间或者 `Scenario` 之间的部分相当于注释掉的描述内容，不会影响测试但是可以作为更具体的说明。
- `Background` : 多个场景如果有相同的部分，就可以写在这里，比如每个场景都需要登录。就可以写在 `Background` 里面。
- `Scenario` ：场景，一个场景可以有一个或多个场景。
- `Given` ：是一个用例开始执行前的一个前置条件，比如打开一个页面。
- `When` ：用例开始执行的一些关键操作步骤，类似点击元素、填写表单。
- `Then` ：观察结果，就是平时用例中的验证步骤。
- `And`/`But`：可以与 `Given`、`When`、`Then` 同时使用，使得 step 描述更清晰易懂

更具体的介绍可以看文档： https://cucumber.io/docs/gherkin/reference/#keywords

### 步骤定义

具体的步骤定义是可以结合自己擅长的语言来实现，例如使用 [Cucumber.js](https://cucumber.io/docs/installation/javascript/) 、 [Cucumber-JVM](https://cucumber.io/docs/installation/java/)、[Cucumber.cpp](https://cucumber.io/docs/installation/cplusplus/) 等，将步骤定义填写完整。

也可以使用支持 cucumber 的框架来填充步骤定义。以下是一个示例：

```bash
npm init
npm install cucumber --save-dev
```

```json
"scripts": {
  "test": "cucumber-js"
},
```

```js
const assert = require("assert");
const { Given, When, Then } = require("cucumber");

function isItFriday(today) {
  return "Nope";
}

Given("today is Sunday", function () {
  this.today = "Sunday";
});

When("I ask whether it's Friday yet", function () {
  this.actualAnswer = isItFriday(this.today);
});

Then("I should be told {string}", function (expectedAnswer) {
  assert.equal(this.actualAnswer, expectedAnswer);
});
```

### 运行测试

```bash
npm test
```

### [生成报告](https://cucumber.io/docs/cucumber/reporting/)

可以加 `--format` 加 formatter 参数来生成相应的测试报告：

- `progress`
- `pretty`
- `html`
- `json`
- `rerun`
- `junit`

例如：

```bash
npm test --format html
```
