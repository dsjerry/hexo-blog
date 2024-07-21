---
title: CSS居中方法
toc: true
date: 2021-04-15 19:31:26
tags: [css]
categories: 前端学习
thumbnail: https://z3.ax1x.com/2021/04/15/cRKzZt.png
cover: https://z3.ax1x.com/2021/04/15/cRKzZt.png
---

收集常见的元素居中方法，正在找，或许还有

左右，上下，上下左右

<!-- more -->

# 上下

## line-height

设置元素的 **_`height`_**和**_`line-height`_** 相等，内容可以得到居中；其实在文本中，值设置 `line-height`，不设置 `height`就可以实现居中了。

- 使用 line-height 实现的垂直居中是近似的居中，上下会差 _1px_ 左右，所以平时也看不出来。

但是，在多行文本中，仅仅使用 line-height 还达不到垂直居中的效果，要加上`vertical-align: middle;`

```css
.box {
  text-align: center;
  line-height: 120px;
}
.content {
  display: inline-block;
  vertical-align: middle;
}
```

## writing-mode

设置元素的 **_`writing-mode`_** 为 `vertical-lr`，然后设置 `text-align: center;`

## margin + transform

```css
.father {
  overflow: hidden;
}
.son {
  margin: 50% auto;
  transform: translateY(-50%);
}
```

# 左右

当元素为 **_`display: inline; display: inline-block;`_** 的时候，**父容器**设置 `text-align: center;` 就可以左右置中；

当元素为 **_`display: block;`_** 的时候，**元素自身**的 `margin-left` 和 `margin-right` 设置为 `auto`，就可以左右置中；

# 上下左右

## position: absolute;

相对于自身设置

```css
position: absolute;
top: 50%;
left: 50%;
transform: translateX(-50%) translateY(-50%);
```

- 设置完 top 和 left 后，元素的左上方会在画布的中心点，然后使用 transform 移动元素自身的一半。
- 也可设置 `right` 和 `bottom`，这时 transform 就不用写负数了。

## position: absolute; 第 2 种

```css
.father {
  position: relative;
}
.son {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
}
```

## flexbox

相对于父级设置

```css
min-height: 100vh;
display: flex;
justify-content: center;
align-item: center;
```

- 100vh 为当前窗口高度。

## display: table-cell;

```css
body{
    display: table;
	width: 100%
	min-height: 100vh;
}
div{
    display: table-cell;
    vertical-align: middle;
    text-align: center;
}
```

## display: grid;

```css
.father {
  display: grid;
}
.son {
  align-self: center;
  justify-self: center;
}
```

## 知道元素尺寸的话

```css
.box {
  width: 100px;
  height: 100px;
  background-color: darkcyan;

  position: relative;
}
.content {
  width: 50px;
  height: 50px;
  background-color: lightblue;

  position: absolute;
  left: 50%;
  top: 50%;
  margin-left: -25px;
  margin-top: -25px;
}
```
