---
title: BFC 块级格式化上下文
toc: true
date: 2021-02-20 15:21:26
tags: [css]
categories: 前端学习
thumbnail: https://s3.ax1x.com/2021/02/20/yI81KS.png
cover: https://s3.ax1x.com/2021/02/20/yI81KS.png
---

**块格式化上下文（Block Formatting Context，BFC）**

相对应还有 内联格式上下文（inline formatting context，IFC），但是影响不太大。

<!-- more -->

# 定义

BFC 是Web页面的可视CSS渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。

一旦一个元素具有 BFC，不管内部元素怎么变化，都不会影响外部的元素。所以会影响外部元素的 *margin合并* 在BFC元素中是不会发生的。

浮动定位和清除浮动时只会应用于同一个BFC内的元素。

## 创建BFC

下面方式会创建块格式上下文

- `<html>`根元素
- 浮动元素（`float`的值不为 none）
- 绝对定位元素（`position`的值是 absolute 或 fixed）
- `overflow`的值不为 visible
- `display: flow-root`，可以**无副作用**创建 BFC
- 表格单元格（元素的 `display`为 table-cell，HTML表格单元格默认为该值）
- 表格标题（`display`为 table-caption，这个也是默认值）
- <a href="https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context">等等</a>

块格式化上下文包含创建它的元素内部的所有内容。

只要元素符合上面的任意一个条件，就**不需要**使用 `clear: both`去清除浮动带来的影响。

# BFC与流体布局

## 例1

BFC 的出现不仅仅是去除 margin 合并和 float 影响，更多是实现更好的自适应效果。

```html
<div class="box">
  <img src="./img/chigua.jpg" />
  <p>好多文字....</p>
</div>
```

```css
img{
    float: left;
}
```

此时文字内容被图片的 *float* 影响，出现了文字环绕

![float影响](https://s3.ax1x.com/2021/02/20/y55G2q.png)

如果给文字加上 BFC 中的任意一个条件，将取消这样的影响，比如加上`overflow: auto`

```css
p { overflow: auto; }
```

![清除影响](https://s3.ax1x.com/2021/02/20/y5vFI0.png)

如果想修改文本和图片之间的间隙

1. 修改浮动元素（这里的图片）的 margin、padding、border 的`right`
2. 修改 BFC 元素的 border 或者 padding

> 注：千万不要使用 BFC 元素的 margin，因为其不会对外界元素造成影响。

## 例2

浮动脱离了文档流，所以 `<div>` 的 `background` 和 `border` 仅仅包含了内容，不包含浮动。

让浮动内容和周围内容等高

```html
<div class="box">
    <div class="float">I am a floated box!</div>
    <p>I am content inside the container.</p>
</div>
```

```css
.box {
    background-color: rgb(224, 206, 247);
    border: 5px solid rebeccapurple;
}
.float {
    float: left;
    width: 200px;
    height: 150px;
    background-color: white;
    border:1px solid black;
    padding: 10px;
}    
```

![上面代码效果](https://s3.ax1x.com/2021/02/20/yI9ebQ.png)

加上任意一个符合 BFC 的条件，比如`overflow: auto`，使得元素变为 BFC 元素

![改变后效果](https://s3.ax1x.com/2021/02/20/yI92ad.png)

更多例子，👉<a href="https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context">MDN 块格式化上下文</a>

## BFC与基于纯流体对比

基于 BFC 实现的自适应布局有点所在

1. 内容因为其封闭性而更加稳定
2. 自适应内容自动填满浮动以外的区域，不用担心浮动元素的宽度

不够将元素 BFC 后也有许多槽点

1. `float: left`。浮动元素本身变成了 BFC，失去了元素本身的流体自适应性。无法用于实现自动填满容器的自适应布局
2. `position: absolute`。大佬，设置了之后，由于独立渲染的原因，别人自己玩去了。
3. `overflow: hidden`。盒子外的元素可能会被隐藏掉。
4. `overflow`的其他属性，可能会出现剪切的阴影或者滚动条。而且在后续开发中回头看，可能会忘掉为什么要使用 overflow，所以用的时候最好加个注释咯。

📖参考：《CSS世界》、<a href="https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context">MDN</a>