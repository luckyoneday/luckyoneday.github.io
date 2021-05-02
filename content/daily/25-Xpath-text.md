---
title: "Xpath的文本匹配"
date: 2020-06-09T21:22:01+08:00
lastmod: 2020-06-09T21:22:01+08:00
draft: false
toc: false
keywords: ["daily"]
description: "Xpath的文本匹配"
author: "youting"
tags: ["Xpath", "e2e"]
---

ummmm 确实很久没有更新过小笔记了...

最近接触了一些新的东西，比如 sketch 插件比如 e2e 测试，可能接下来会穿插着这两项的小知识点记一下小笔记~

今天是一篇关于 e2e 测试的一篇翻译：Xpath 中文本匹配方式，来自 [stackOverflow](https://stackoverflow.com/questions/3655549/xpath-containstext-some-string-doesnt-work-when-used-with-node-with-more)。

首先为什么要使用 Xpath 呢，首先是因为我们的项目中有用 css-modules 的，有用 styled-component 的，className 首先不确定，也不想为每一个元素都添加 `data-*` 属性；其次相对来说文案是比较固定且通俗易懂的，Xpath 中的 `text()` 为依赖文案的测试提供了便利，并且支持多属性组合、父元素子孙元素搜索，功能比较强大。

翻译从这里开始：

现下有一个 XML 结构如下：

```xml
<Home>
  <Addr>
    <Street>ABC</Street>
    <Number>5</Number>
    <Comment>BLAH BLAH BLAH <br /><br />ABC</Comment>
  </Addr>
</Home>
```

想获取到当前结构中所有包含 ABC 文本的节点，使用 `//*[contains(text(), "ABC")]` 只匹配到了 `<Street>` 但是没有匹配到 `<Comment>` 。是因为什么呢？

我们需要了解一下 Xpath 的匹配方式， `//*[contains(text(), "ABC")]` 中，大致是以下匹配规则：

1. `*` 是一个选择器，会匹配所有的元素 -- 会返回一个节点集（node-set）
2. `[]` 是在该节点集中每个节点上运行的条件，若当前节点匹配括号内的条件则匹配，返回布尔值，对第一步的节点集进行过滤操作
3. `text()` 是一个选择器，是拿到当前节点的所有文本子节点
4. `contains` 是对字符串进行运算的函数，它的两个参数都是字符串，如果参数不是字符串则会调用 `string()` 方法转换为字符串，所以会把 `<Comment>` 的文本子节点转换为字符串， `string()` 方法对文档顺序的[第一个节点](https://www.w3.org/TR/xpath-10/#function-string)返回 `string-value`：
   - 对于 [元素节点](https://www.w3.org/TR/xpath-10/#element-nodes) 通过返回按文档顺序首先出现的节点集中节点的字符串值，可以将节点集转换为字符串。如果节点集为空，则返回一个空字符串。
   - 对于 [文本节点](https://www.w3.org/TR/xpath-10/#section-Text-Nodes) 通过返回文本节点的字符串值将节点转换为字符串。

所以这个时候当前文本节点是第一个文本子节点，即 `BLAH BLAH BLAH` ，并不包含 `ABC`，所以没有匹配到 `<Comment>`

那怎么解决这个问题呢，可以这样写 `//*[text()[contains(.,"ABC")]]`:

1. `*` 是一个选择器，会匹配所有的元素 -- 会返回一个节点集（node-set）
2. `[]` 是在该节点集中每个节点上运行的条件 -- 在此，它对文档中的每个元素进行操作
3. `text()` 是一个选择器，是拿到当前节点的所有文本子节点
4. 内部的`[]` 是在该节点集中每个节点上运行的条件 -- 这里是文本节点
5. `contains` 是对字符串进行运算的函数，这里的 `.` 指的是对当前的文本节点进行匹配

此时会对文本节点进行依次匹配，而不仅是第一个文本子节点了，就可以匹配到 `<Street>` 和 `<Comment>`。

so，`//*[contains(.，"ABC")]` 匹配包含 `ABC` 的任何元素（但根节点除外）。对于上面的 XML 文档，它与 `<Home>`，`<Addr>`，`<Street>` 和 `<Comment>` 元素匹配。同理，`//*[contains(.，"BLAH ABC")]` 与 `<Home>`，`<Addr>` 和 `<Comment>` 元素匹配。
