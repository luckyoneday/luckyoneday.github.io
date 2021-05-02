---
title: "随滚轮滚动的简单实现"
date: 2019-12-18T15:03:24+08:00
lastmod: 2019-12-18T15:03:24+08:00
draft: false
toc: false
react: true
keywords: ["daily"]
description: "随滚轮滚动的简单实现"
author: "youting"
tags: ["React", "Event", "JavaScript"]
---

<style>
#root {
  width: 100%;
  height: 240px;
  border: 2px solid #f1f1f1;
}

.wrapper {
  width: 100%;
  height: 100%;
}

.inner {
  position: absolute;
  z-index: 10;
  width: 100px;
  height: 100px;
  transform-origin: 0 0;
  background: #1199ee;
}

</style>

效果见蓝色方块。（移动端不支持）

<div id="root"></div>

这里就要用到[之前提到过](/daily/2019-12-17/)的`wheelEvent`：

```js
// ...

useEffect(() => {
  const handleWheel = (e) => {
    // 拿初始位置
    const left = nodePosRef.current.x;
    const top = nodePosRef.current.y;

    // 鼠标滚动的距离
    const diffX = e.deltaX;
    const diffY = e.deltaY;

    // 目标位置
    const targetNodeX = left - diffX;
    const targetNodeY = top - diffY;

    setStyle({ x: targetNodeX, y: targetNodeY });

    // 记录上一次的位置
    nodePosRef.current.x = targetNodeX;
    nodePosRef.current.y = targetNodeY;
  };

  window.addEventListener("wheel", handleWheel);
  return () => {
    window.removeEventListener("wheel", handleWheel);
  };
}, []);
```

<script type="text/babel">

const rootElement = document.getElementById("root");

const useScroll = () => {
  const nodeRef = React.useRef(null);
  const nodePosRef = React.useRef({ x: 0, y: 0 });

  const setStyle = ({ x, y }) => {
    if (nodeRef.current) {
      nodeRef.current.style.transform = 'translate(' + x + 'px,' + y + 'px)';
    }
  };

  React.useEffect(() => {
    const handleWheel = e => {
      e.preventDefault();

      const left = nodePosRef.current.x;
      const top = nodePosRef.current.y;

      const diffX = e.deltaX;
      const diffY = e.deltaY;

      const targetNodeX = left - diffX;
      const targetNodeY = top - diffY;

      setStyle({ x: targetNodeX, y: targetNodeY });

      nodePosRef.current.x = targetNodeX;
      nodePosRef.current.y = targetNodeY;
    };

    rootElement.addEventListener("wheel", handleWheel, { passive: false });
    return () => {
      rootElement.removeEventListener("wheel", handleWheel, { passive: false });
    };
  }, []);

  return node => (nodeRef.current = node);
};

function App() {
  const getNode = useScroll()
  return (
    <div className="wrapper">
      <div className="inner" ref={getNode}> 
      </div>
    </div>
  );
}

ReactDOM.render(<App />, rootElement);
</script>
