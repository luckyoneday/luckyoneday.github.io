---
title: "Wheel Event"
date: 2019-12-17T17:21:52+08:00
lastmod: 2019-12-17T17:21:52+08:00
draft: false
keywords: ["daily"]
description: "滚轮滚动的事件"
author: "youting"
tags: ["React", "Event", "JavaScript"]
---

和[`scroll`](https://developer.mozilla.org/en-US/docs/Web/API/Document/scroll_event)不同的是，`scroll`是监听页面或元素滚动，`wheel`是监听滚轮滚动。`wheelEvent`的默认操作是基于浏览器具体实现的，而且不一定会触发“滚动”这个行为，即使它触发了，`wheel`的`delta*`属性也不一定代表了滚动方向。so 不要依赖`delta*`属性来获取页面滚动方向，应该在`scroll`事件中检测目标的[`scrollLeft`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollLeft)和[`scrollTop`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollTop)的值来判断方向。

```js
target.addEventListener("wheel", handleWheel);
target.onwheel = handleWheel;
```

这个事件继承了父接口 [MouseEvent](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent)、[UIEvent](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent)、[Event](https://developer.mozilla.org/en-US/docs/Web/API/Event)的属性，除此之外还有四个属性：

- [`deltaX`](https://developer.mozilla.org/en-US/docs/Web/API/WheelEvent/deltaX): 水平方向滚动距离
- [`deltaY`](https://developer.mozilla.org/en-US/docs/Web/API/WheelEvent/deltaY): 垂直方向滚动距离
- [`deltaZ`](https://developer.mozilla.org/en-US/docs/Web/API/WheelEvent/deltaZ): z 轴滚动距离
- [`deltaMode`](https://developer.mozilla.org/en-US/docs/Web/API/WheelEvent/deltaMode): delta 滚动值的单位

| Constant        | Value | Description             |
| --------------- | ----- | ----------------------- |
| DOM_DELTA_PIXEL | 0x00  | delta 的值为 像素 级别. |
| DOM_DELTA_LINE  | 0x01  | delta 的值为 行 级别.   |
| DOM_DELTA_PAGE  | 0x02  | delta 的值为 页 级别.   |

在 React 中，这个事件叫做[`onWheel`](https://reactjs.org/docs/events.html#wheel-events)。

## [`passive event listener`](https://github.com/WICG/EventListenerOptions/blob/gh-pages/explainer.md)

在做 wheel + ctrl 触发元素缩放时，在谷歌浏览器中会触发页面整体放大，理所当然的认为`preventDefault()`可以阻止这个默认行为，但是触发了一个报错：

`[Intervention] Unable to preventDefault inside passive event listener due to target being treated as passive. See`[`https://www.chromestatus.com/features/6662647093133312`](https://www.chromestatus.com/features/6662647093133312)

大致意思是，如果未另行指定，则在文档级目标(window.document，window.document.body 或 window)上注册的 wheel / mousewheel 事件将被视为被动的(passive)，并且在此类事件中调用`preventDefault()`将会被忽略。

可以这样解决：

```js
target.addEventListener("wheel", handleWheel, { passive: false });
```

一般来说，[`addEventListener`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)的第三个参数是`useCapture`，表示是否在捕获阶段调用`event handler`，默认为`false`。但是其实也可以是一个`options`：

```js
target.addEventListener(type, listener, options);
target.addEventListener(type, listener, useCapture);
```

`options`的三个常用属性：

- `capture`: 表示事件是否在捕获阶段调用`event handler`
- `once`: 表示`listener`在添加之后最多只调用一次。如果是`true`，`listener`会在其被调用之后自动移除。
- `passive`: 设置为`true`时，表示`listener`永远不会调用`preventDefault()`。如果仍然调用了`preventDefault()`，客户端将会忽略它并抛出一个控制台警告。

`passive`设置为`true`有什么好处呢！这里有个[解释](https://developers.google.com/web/updates/2016/06/passive-event-listeners?hl=zh-cn)，意思是浏览器无法预先知道一个`event handler`会不会调用`preventDefault()`，它需要等`handler`的执行，才能知道是否执行默认行为，而`handler`执行是要耗时的，这时再去执行默认行为就会导致页面卡顿。而`passive`设置为`true`是告诉浏览器不会调用`preventDefault()`，不需要等待`handler`执行完就先按照默认行为进行下去，从而提高滑动或滚动事件的流畅度。

> 自我感觉意思就是当不需要阻止页面的默认行为时，设置`{ passive: true }`会对体验有很大的优化，而当真的不需要页面的默认行为时，设置`{ passive: false }`和`preventDefault()`是没问题的。

这里还有一篇更加深层的文章：[让页面滑动流畅得飞起的新特性：Passive Event Listeners](https://cloud.tencent.com/developer/article/1004401)
