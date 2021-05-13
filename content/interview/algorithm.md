---
title: "一些算法"
date: 2021-04-17T12:09:44+08:00
lastmod: 2021-04-17T12:09:44+08:00
draft: false
toc: false
keywords: ["interview"]
description: "一些算法"
tags: ["interview"]
author: "youting"
---

## 排序算法

### 冒泡排序

比较相邻的元素，如果第一个比第二个大，就交换\-\-\-\-这样会找出最大的元素，随后对除最后一个元素进行同样的操作。

## 链表

### 反转链表

### 快慢指针算法

- 可用来判断链表是否有环，和环入口在哪里
- 可用来获取链表的中间节点、或者是倒数第几个节点

## 树

仅有前序和后序遍历，不能确定一个二叉树，必须有中序遍历的结果。

### 前中后序遍历

二叉树的前中后序遍历，使用递归算法实现最为简单：

```js
var postorderTraversal = function (root, result = []) {
  if (root) {
    postorderTraversal(root.left, result);
    postorderTraversal(root.right, result);
    // 前中后序 -- 递归解法区别只在这一行的位置
    result.push(root.val);
  }

  return result;
};
```

非递归遍历使用栈存储节点，记录经过的节点：

{{% admonition type="info" title="前序遍历" details="true" %}}

```js
var preorderTraversal = function (root) {
  if (!root) return [];
  const stack = [];
  const result = [];

  let p = root;

  while (p || stack.length) {
    if (p) {
      stack.push(p);
      result.push(p.val);
      p = p.left;
    } else {
      p = stack.pop();
      p = p.right;
    }
  }

  return result;
};
```

{{% /admonition %}}

{{% admonition type="info" title="中序遍历" details="true" %}}

```js
var inorderTraversal = function (root) {
  if (!root) return [];
  const stack = [];
  const result = [];
  let p = root;

  while (p || stack.length) {
    if (p) {
      stack.push(p);
      p = p.left;
    } else {
      p = stack.pop();
      // 和前序遍历的区别：只有 push value 的地方不一样
      result.push(p.val);
      p = p.right;
    }
  }

  return result;
};
```

{{% /admonition %}}

{{% admonition type="info" title="后序遍历" details="true" %}}

需要一个指针指向上一次出栈的元素，如果当前节点的 `right` 指针为 null，或者 `right` 已经出栈则表示可以出栈。

```js
var postorderTraversal = function (root) {
  if (!root) return [];
  const stack = [];
  const result = [];
  let p = root,
    lastOne = null;

  while (p || stack.length) {
    if (p) {
      stack.push(p);
      p = p.left;
    } else {
      const topOne = stack[stack.length - 1];

      if ((topOne.right === null || topOne.right = lastOne)) {
        stack.pop();
        result.push(topOne.val);
        lastOne = topOne;
      } else {
        p = topOne.right;
      }
    }
  }

  return result;
};
```

{{% /admonition %}}

### 层次遍历

分为深度优先（DFS）和广度优先（BFS）。

## 动态规划
