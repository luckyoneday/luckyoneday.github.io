---
title: "CSS面试知识点汇总"
date: 2021-05-01T13:54:32+08:00
lastmod: 2021-05-01T13:54:32+08:00
draft: false
toc: true
keywords: ["interview"]
description: "CSS面试知识点汇总"
tags: ["interview"]
author: "youting"
---

## CSS 优化

- 内联首屏关键 css

- 异步加载非首屏 css

  - 使用 javascript 将 link 标签查到 head 标签最后；
  - 设置 `link` 标签 `media` 属性为 `noexist`，浏览器会认为当前样式表不适用当前类型，会在不阻塞页面渲染的情况下再下载。加载完成后再将 `media` 值设置为 `screen` 或 `all`，从而让浏览器开始加载 css： `<link rel="stylesheet" href="mystyles.css" media="noexist" onload="this.media='all'" />`；
  - 通过 `rel` 属性将 `link` 标记为 `alternate` 可选样式表，也可以实现浏览器异步加载。加载完成后，将 `rel` 设置回 `stylesheet`： `<link rel="alternate stylesheet" href="mystyles.css" onload="this.rel='stylesheet'" />`；

- 资源压缩；

- 合理使用选择器：

  - 不要嵌套使用过多复杂选择器；不要三层以上；
  - 使用 id 选择器没有必要进行嵌套；
  - 通配符和属性选择器效率较低，尽量避免使用；

- cssSprite，合成所有 icon 图片，用宽高加上 `background-position` 的背景图方式显现 icon 图，减少 http 请求;

- 把小的 icon 图片转成 base64 编码;

## BFC

块级格式化上下文。触发条件：

- 根元素，即 HTML 元素

- 浮动元素，`float` 为 `left`、`right`

- `overflow` 不为 `visible`，为 `auto`、`scroll`、`hidden`

- `display` 为 `inline-block`、`inltable-cell`、`table-caption`、`table`、`inline-table`、`flex`、`inline-flex`、`grid`、`inline-grid`

- `position` 的值为 `absolute` 或 `fixed`

应用场景：

- 防止 `margin` 塌陷

- 清除内部浮动

- 自适应多栏布局

## 一些单位

- em：相对长度单位，相对于当前对象内文本的字体尺寸（针对父元素的尺寸）；如当前行内文本的字体尺寸未被人为设置，则相对于浏览器的默认字体尺寸 `16px`；

- px：与设备像素比有关，但是与元素的其他属性无关；

  - 设备像素比（DPR）：`window.devicePixelRatio`

- rem：相对长度单位，相对的是 `HTML` 根元素 `font-size` 的值；

- vh、vw：相对于窗口宽高，100vw 代表满宽，100vh 表示满高；

## 响应式布局

响应式网站设计是一种网络页面设计布局，页面的设计与开发应当根据用户行为以及设备环境（系统平台、屏幕尺寸、屏幕定向等）进行相应的响应和调整。

原理是通过媒体查询检测不同的设备屏幕尺寸并进行处理。

### 媒体查询

`html` 中需要添加 `<meta>` 标签来声明 `viewport。`

```html
<meta
  name="viewport"
  content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no"
/>
```

css 中使用 @media 查询，可以针对不同的媒体类型定义不同的样式：

```css
@media screen (min-width: 375px) and (max-width: 600px) {
  body {
    font-size: 18px;
  }
}
```

## 居中布局

块状元素：

- 利用 `position` + `margin: auto`：需要子元素 `top/right/bottom/left` 设为相同的值，然后使用 `margin: auto`；

- 利用 `position` + `margin: 负值`：需要子元素 `top: 50%; left: 50%; margin-left: 负一半自身宽度; margin-top: 负一半自身高度`，需要知道自身宽高；

- 利用 `position` + `transform`：需要子元素 `top: 50%; left: 50%; transform: translate(-50%, -50%)`，不需要知道自身宽高；

- table 布局：父元素使用 `display: table-cell;text-align: center;vertical-align: middle` ，子元素使用 ` display: inline-block`；

- flex 布局：父元素使用 `display: flex;justify-content: center;align-items: center`；

- grid 布局：父元素使用 `display: grid;align-items: center;justify-content: center`

行内元素：

- `text-align: center`; `height` = `line-height`

## CSS3 新增的特性

### 新增选择器

- `:nth-child(n)`
- `:first-of-type`
- `:last-child`
- `:disabled`

### 边框

- `border-radius`
- `box-shadow`
- `border-image`

### 背景

- `background-clip`
- `background-origin`
- `background-size`
- `background-break`

### 文字

- `word-wrap`
- `text-overflow`
- `text-shadow`
- `text-decoration`

单行文本溢出显示省略号：

```css
overflow: hidden;
white-space: nowrap;
text-overflow: ellipsis;
```

多行文本溢出显示省略号：

```css
-webkit-line-clamp: 2;
display: -webkit-box;
-webkit-box-orient: vertical;
overflow: hidden;
text-overflow: ellipsis;
```

### transition

```css
transition-property: width;
transition-duration: 1s;
transition-timing-function: linear;
transition-delay: 2s;
```

### transform

不支持 inline 元素。

`transform-origin` 修改 transform base 的位置。

```css
transform: translate(120px, 50%); /* 位移 */
transform: scale(2, 0.5); /* 缩放 */
transform: rotate(0.5turn); /* 旋转 */
transform: skew(30deg, 20deg); /* 倾斜 */
```

### animation

```css
animation-name：name;
animation-duration：动画持续时间
animation-timing-function：动画时间函数
animation-delay：动画延迟时间
animation-iteration-count：动画执行次数，可以设置为一个整数，也可以设置为infinite，意思是无限循环
animation-direction：动画执行方向
animation-play-state：动画播放状态
animation-fill-mode：动画填充模式，backwards、forwards

@keyframe name {
 from {
 }
 to {
 }
}
```

### 渐变

- `linear-gradient`
- `radial-gradient`

### flex

容器属性：

- `flex-direction`：主轴方向，`row` | `row-reverse` | `column` | `column-reverse`

- `flex-wrap`：是否可换行，`nowrap` | `wrap` | `wrap-reverse`

- `flex-flow`：是 `flex-direction` 属性和 `flex-wrap` 属性的简写形式，默认值为 `row nowrap`

- `justify-content`： `flex-start` | `flex-end` | `center` | `space-between` | `space-around`

- `align-items`：`flex-start` | `flex-end` | `center` | `baseline` | `stretch`

- `align-content`：`flex-start` | `flex-end` | `center` | `space-between` | `space-around` | `stretch`

成员属性：

- `order`：定义项目的排列顺序，数值越小，排列越靠前，默认为 0

- `flex-grow`：定义项目的放大比例

- `flex-shrink`：定义项目的缩小比例

- `flex-basis`：定义初始尺寸

- `flex`：是 `flex-grow`，`flex-shrink` 和 `flex-basis` 的简写

  - `flex: 1 = flex: 1 1 0%`
  - `flex: 2 = flex: 2 1 0%`
  - `flex: auto = flex: 1 1 auto`
  - `flex: none = flex: 0 0 auto` 常用于固定尺寸不伸缩

- `align-self`：允许单个项目有与其他项目不一样的对齐方式，可覆盖 `align-items`

### grid

容器属性：

- `display`：`grid` | `inline-grid`，标识容器是块级元素还是行内元素

- `grid-template-columns`：设置列宽；`grid-template-rows`：设置行高

  - 可以使用 `repeat()` 函数，如 `repeat(3, 100px)`
  - `auto-fill`：如 `repeat(auto-fill, 200px)`，表示可以放下几个就放几个
  - `fr`：片段，表示比例关系，`grid-template-columns: 200px 1fr 2fr` 表示第一个列宽设置为 `200px`，后面剩余的宽度分为两部分，宽度分别为剩余宽度的 1/3 和 2/3

- `grid-row-gap`：设置行间距；`grid-column-gap`：设置列间距；`grid-gap`：是前两个的缩写；

- `grid-template-areas`：用于定义区域

  - `grid-template-areas: "a b c" "d e f" "g . ."`：分成 7 个区域，`.` 表示无用区域
  - `grid-template-areas: 'a a a' 'b b b' 'c c c'`：分成 `a b c` 三个区域

- `grid-auto-flow`：`row` | `column`，标识放置顺序，先行后列或是先列后行

- `justify-items`：设置单元格内容的水平位置；`align-items`：设置单元格的垂直位置

  - `start` | `end` | `center` | `stretch`
  - `place-items`：是合并简写形式

- `justify-content`：设置整个内容区域在容器的水平位置；`align-content`：设置整个内容区域在容器的垂直位置

  - `start` | `end` | `center` | `stretch` | `space-around` | `space-evenly`
  - `place-content`：是合并简写形式

- `grid-auto-columns`：设置隐形网格的高度；`grid-auto-rows`：设置隐形网格的宽度；

成员属性：

- `grid-column-start`：左边框所在的垂直网格线；`grid-column-end`：右边框所在的垂直网格线；`grid-row-start`：上边框所在的垂直网格线；`grid-row-end`：下边框所在的垂直网格线；

- `grid-area`：定义项目放置在哪个区域，这里的区域是上面的 `grid-template-area`

- `justify-self`：设置单个项目在容器的水平位置；`align-self`：设置单个项目在容器的垂直位置

  - `start` | `end` | `center` | `stretch`
  - `place-self`：是合并简写形式
  - 覆盖父元素的 `justify-items` 和 `align-items`

## 其他的一些

- 视差滚动 `background-attachment: fixed`

- 吸顶 `position: sticky`

## 示例

- [BFC](https://codesandbox.io/s/bfc-vvvlp)
- [居中](https://codesandbox.io/s/center-d8l0c)
- [动画](https://codesandbox.io/s/animate-g4u4h)
- [flex](https://codesandbox.io/s/flex-ure85)
- [grid](https://codesandbox.io/s/grid-8fb3v)
