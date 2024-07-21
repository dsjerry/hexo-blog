---
title: CSS伪元素中before和after
toc: true
date: 2021-02-18 19:42:56
tags: [css]
categories: 前端学习
thumbnail: https://s3.ax1x.com/2021/02/18/yWW9wq.png
cover: https://s3.ax1x.com/2021/02/18/yWW9wq.png
---

主要内容：介绍伪元素、伪元素其中的 `::before` 和 `::after`

参考内容：<a href="https://developer.mozilla.org/zh-CN/docs/Web/CSS/::before">MDN</a>

<!-- more -->

# 伪元素

伪元素是一个附加至选择器末的关键词，允许你对被选择元素的特定部分修改样式。-- MDN

语法

```css
selector::pseudo-element {
  property: value;
}
```

- 一个选择器只能使用一个伪元素。为区分伪类和伪元素，按照规范使用`::`代表伪元素，不过`:`也支持。

# ::before

创建一个伪元素，其将成为匹配选中的元素的**第一**个子元素。通常通过 `content` 属性来为元素添加修饰性的内容。

- 元素默认是行内元素
- **不适合用在替换元素上**

## 语法

```css
/* CSS3 语法 */
element::before { 样式 }

/* （单冒号）CSS2 过时语法 (仅用来支持 IE8) */
element:before  { 样式 }

/* 在每一个p元素前插入内容 */
p::before { content: "Hello world!"; } 
```

## 案例

1. todo list 前面的对勾

直接戳👉<a href="https://developer.mozilla.org/zh-CN/docs/Web/CSS/::before">MDN CSS ::before</a>

# ::after

创建一个伪元素，其将成为匹配选中的元素的**最后**一个子元素。通常通过 `content` 属性来为元素添加修饰性的内容。

- 元素默认是行内元素
- **不适合用在替换元素上**

## 语法

```css
element:after  { style properties }  /* CSS2 语法 */

element::after { style properties }  /* CSS3 语法 */
```

> **注:** IE8仅支持`:after。`

## 案例

1. 简单用法：在内容后面加一些注释
2. 装饰用法：用想要的任何方法给content属性里的文字和图片的加上样式
3. 提示用法：用作 hover 之后的提示

直接戳👉<a href="https://developer.mozilla.org/zh-CN/docs/Web/CSS/::after">MDN CSS ::after</a>

# ::before 和 ::after 画图

## 六边形

```html
<div id="six"></div>
```

```css
#six {
  width: 0;
  height: 0;
  border-left: 50px solid transparent;
  border-right: 50px solid transparent;
  border-bottom: 100px solid red;
  position: relative;
}
#six::after {
  width: 0;
  height: 0;
  border-left: 50px solid transparent;
  border-right: 50px solid transparent;
  border-top: 100px solid red;
  position: absolute;
  content: "";
  top: 30px;
  left: -50px;
}
```

## 小图标



待补充.....





















