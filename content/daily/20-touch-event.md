---
title: "Touch Event"
date: 2019-12-22T10:19:17+08:00
lastmod: 2019-12-22T10:19:17+08:00
draft: false
keywords: ["daily"]
description: "触摸事件-touch"
author: "youting"
tags: ["React", "Event", "JavaScript"]
---

[`TouchEvent`](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events)是一类描述手指在触摸屏面（触摸屏、触摸板）等状态变化的事件。属于一种[`UIEvent`](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent)。该事件可以描述与屏幕的一个或多个触点，并且支持检测移动，触点的添加和移除。

每个触点都是一个`Touch`对象，每个[`Touch`](https://developer.mozilla.org/en-US/docs/Web/API/Touch)对象都有位置、尺寸、形状、压力大小和目标元素等属性，触点列表由[`TouchList`](https://developer.mozilla.org/en-US/docs/Web/API/TouchList)对象表示。

- [`TouchEvent.changedTouches`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/changedTouches): 是一个`TouchList`对象，列出了这次触摸事件中发生了改变的`Touch`对象。

  - 对于`touchstart`事件，这个对象是上次事件中新增加的激活的触点的列表。
  - 对于`touchmove`事件，这个对象是与上次事件相比发生了变化的触点列表。
  - 对于`touchend`事件，这个对象是从触屏中移除的触点的列表。

- [`TouchEvent.targetTouches`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/targetTouches): 是一个只读的`TouchList`列表，包含仍与目标对象接触的所有触摸点的`Touch`对象。
- [`TouchEvent.touches`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/touches): 当前在与触摸表面接触的`Touch`对象，不管触摸点是否已经改变或其目标元素是在处于哪个事件阶段。

## touch 事件的类型

- [`touchstart`](https://developer.mozilla.org/en-US/docs/Web/API/Element/touchstart_event): 当用户在触摸设备上放置了一个触点时触发。
- [`touchmove`](https://developer.mozilla.org/en-US/docs/Web/API/Document/touchmove_event): 当用户在触摸设备上移动触点时触发，当触点的半径、旋转角度以及压力大小发生变化时，也将触发此事件。
- [`touchend`](https://developer.mozilla.org/en-US/docs/Web/API/Document/touchend_event): 当一个触点被用户从触摸平面上移除（将一个手指离开触摸平面）时触发。当触点移出触摸平面的边界时也将触发。例如用户将手指划出屏幕边缘。
