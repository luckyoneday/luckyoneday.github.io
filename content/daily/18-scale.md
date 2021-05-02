---
title: "鼠标为中心缩放的简单实现"
date: 2019-12-20T21:52:47+08:00
lastmod: 2019-12-20T21:52:47+08:00
draft: false
toc: false
keywords: ["daily"]
description: "鼠标为中心缩放的简单实现"
author: "youting"
mathjax: true
tags: ["React", "Event", "JavaScript"]
---

> 最终效果:
>
> ![useScale](../../static-img/useScale.gif)

```js
// ...
useEffect(() => {
  const handleScale = (e) => {
    // 这里用到了阻止默认行为
    e.preventDefault();

    const cursorX = e.clientX;
    const cursorY = e.clientY;

    // == 这里以下都是纯计算逻辑啦
    const posRateX = (cursorX - initLeft) / initWidth;
    const posRateY = (cursorY - initTop) / initHeight;

    const currentHeight = initHeight - e.deltaY;

    // 保证 width 和 height 变化的比例一样
    const currentWidth = initWidth * (currentHeight / initHeight);

    const diffX = currentWidth * posRateX;
    const diffY = currentHeight * posRateY;

    const targetNodeX = cursorX - diffX;
    const targetNodeY = cursorY - diffY;

    setStyle({
      x: targetNodeX,
      y: targetNodeY,
      width: currentWidth,
      height: currentHeight,
    });
    // 再记录位置和尺寸参数
    // ...
  };

  // 这里传了第三个参数 `{ passive: false }`
  window.addEventListener("mousewheel", handleScale, {
    passive: false,
  });
  return () => {
    window.removeEventListener("mousewheel", handleScale, {
      passive: false,
    });
  };
}, []);
```

这里用的计算方法是比如 y 轴方向上：

$$\frac{cursorY - initTop}{initHeight} = \frac{cursorY - currentTop}{currentHeight}$$
