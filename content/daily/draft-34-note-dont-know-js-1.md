---
title: "笔记 -- 你不知道的JavaScript 1"
date: 2020-08-18T11:31:41+08:00
lastmod: 2020-08-18T11:31:41+08:00
draft: true
toc: true
keywords: ["daily"]
description: "你不知道的JavaScript的读书笔记"
tags: ["笔记", "你不知道的JavaScript", "JavaScript"]
author: "youting"
---

## 编译原理

传统编译语言的流程中，程序中的一段源代码在执行之前会经历三个步骤，统称为“编译”。

- 分词 / 词法分析 (Tokenizing / Lexing)

  将字符串分解为词法单元（有意义的代码块），「分词」和「词法分析」的区别在于词法单元的识别是通过有状态还是无状态的方式进行的。有状态的解析规则是「词法分析」，否则就是「分词」。

- 解析 / 语法分析 (Parsing)

  将词法单元解析成「抽象语法树」(Abstract Syntax Tree, AST)，可以在 [AST explorer](https://astexplorer.net/)查看解析效果。

- 代码生成

  将 AST 转换为可执行代码的过程被称为代码生成。
