---
title: 移动web开发
toc: true
date: 2021-07-21 16:47:40
tags: [移动端]
categories: 前端学习
thumbnail: https://z3.ax1x.com/2021/08/06/fnRc0e.png
cover: https://z3.ax1x.com/2021/08/06/fnRc0e.png
---

简单介绍移动端 web 开发所需要的基础知识

好好学习。<!-- more -->

# 基础

## `<meta>`视口标签

```html
<meta
  name="viewport"
  content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0"
/>
```

添加视口标签之前，页面会有拖动条，这是默认的页面大小。添加视口标签之后，页面就会自动适应设备的大小，拖动条也不会出现了。

## 二倍图

单位`px`和真实的物理像素（分辨率）在移动端是有一定程度的偏差的，往往是`1px`对应多个像素点

例：需要在屏幕显示`50px`的图片，放到手机屏幕可能会被放大到`100px`，这样图片就会模糊。准备一张`100px`的图片，然后手动调节这张图片的 CSS 样式为`50px`，以此解决模糊。

不仅是二倍图，叶可能是三倍图、四倍图。

对于背景图片的设置，使用`background-size`属性，其值除了数值单位，还有*cover*和*contain*。后两者的区别是*cover*铺满整个容器，*contain*的宽或者高铺满容器后就停止缩放。

## 开发选择

1. 单独的移动端页面：为移动端另外写代码（推荐），通过打开网站的设备来跳转到不同的页面。
2. 响应式页面兼容移动端：一套代码包含桌面和移动端（不太推荐），通过判断屏幕的宽度来改变样式。

## 其他

1. CSS 初始化——<a href="https://github.com/necolas/normalize.css/">normalize.css</a>。`npm install --save normalize.css`
2. 可以使用`box-sizing`属性解决一些问题，省去减去`margin、padding`的麻烦
3. 对于 Safari 中的标准，清除类似于`<a>`点击的高亮效果：`-webkit-tap-heighlight-color: transparent`（非标准）
4. 对于 Safari 上按钮和输入框自定义样式添加前提：`-webkit-apperance: none`（实验中的功能）
5. 对于 Safari 上，如`<a>`，按住后会弹出这个跳转的预览窗口，`-webkit-touch-callout: none`可以关闭它（非标准）(仅支持 Safari)

**单独页面常见布局**：流式布局、flex 布局、less+rem+媒体查询布局、混合布局
**响应式页面兼容**：媒体查询、Bootsrap 之类的框架

# 流式布局

- 也称*百分比布局*、_非固定像素布局_
- 一般会使用到`float`、`position`

## 基础样式

基本宽度

```css
body {
  width: 100%;
  min-width: 320px;
  max-width: 640px;
  margin: 0 auto;
}
```

然后接下来的内容的宽度都是使用`%`作为单位。

**例 1**，两边绿色的是固定位置固定大小的按钮，中间的是搜索框，长度可以随着屏幕尺寸的改变而改变

![搜索框案例](https://z3.ax1x.com/2021/07/22/WBaSR1.png)

```css
.main {
  position: relative;
  height: 50px;
}
.left {
  position: absolute;
  top: 0;
  left: 0;
  width: 50px;
  height: 50px;
}
.right {
  position: absolute;
  top: 0;
  right: 0;
  width: 50px;
  height: 50px;
}
/* 中间的是标准流就行，会铺满。然后设置margin或者padding调节左右间隙的距离 */
.center {
  height: 50px;
  margin: 0 50px;
}
```

- 注：固定定位的盒子要有个宽度才行。

**例 2**，一个导航栏里面有 10 个图标，每行 5 个。可以直接使用`<nav>`和`<a>`，`<a>`里面放具体内容

![例2](https://z3.ax1x.com/2021/07/25/W2itdP.png)

```css
.nav a {
  float: left;
  width: 20%;
}
```

## 精灵图

- 以`background`的形式存在

对于二倍精灵图，在图片编辑工具中将精灵图**等比**缩放到原来的**一半**，然后根据大小，测量出所需图标在整个精灵图中所处的坐标，如：`background: url(./img/jl.png) no-repeat 100px 0`，后面的就是坐标。

> 在图片编辑器中缩放只是作为坐标参考，不要保存缩放后的图片。

最后，还要使用`background-size`来在 CSS 层面缩小精灵图为原来的**一半**

# flex 布局

- 任何一个容器都可以使用`flex`布局
- 父盒子设置为`flex`的时候，`float、clear、vertical-align`失效

**常见父项属性**

| 属性            | 描述                                                                 |
| --------------- | -------------------------------------------------------------------- |
| flex-direction  | 设置**主轴**的方向                                                   |
| justify-content | 设置<span style="color: orange;">主轴</span>上子元素排列方式         |
| flex-wrap       | 设置子元素是否换行                                                   |
| align-content   | 设置<span style="color: orange;">侧轴</span>上子元素排列方式（多行） |
| align-items     | 设置侧轴上子元素排列方式（单行）                                     |
| flex-flow       | 相当于同时设置了 flex-direction 和 flex-wrap                         |

- 元素是跟着主轴来排列的，所以先知道是哪一条

`flex-wrap`，是否换行。各个子元素在主轴方向上的总长度**大于**父元素的长度，使用`wrap`之后会换行（并且一行或一列显示多少个，取决于子元素的长度）。如果不使用，子元素的宽度就会等比缩小，挤在一行或是一列。

`align-item`，其中`stretch`值是拉伸效果，前提是子元素**没有设置高度**

`align-content`，多了`space-around、space-between`，可以设置上一行置顶，下一行在最底（前提是要有换行）。`align-item`是不行的，因为只能操作一行。

**常见子项属性**

| 属性       | 描述                                                     |
| ---------- | -------------------------------------------------------- |
| flex       | 子项占的份数（不仅能左右还能上下，所以比百分比好用些）   |
| align-self | 控制子项自己在侧轴的排列方式（某单个）                   |
| order      | 属性定义子项的排列顺序（前后排序，默认是 0，可以为负数） |

`flex`的值可以为百分数。

## 基础样式

对于顶固的搜索框，应该是固定定位的，并且固定定位位置与父元素无关，是以屏幕为准的

```css
.search {
  position: fixed;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  /* 固定的盒子要有宽度 */
  width: 100%;
  min-width: 320px;
  max-width: 540px;
  height: 50px;
}
```

📋 参考：黑马程序员
