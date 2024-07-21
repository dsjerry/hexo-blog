---
title: js生成验证码
toc: true
date: 2021-02-09 15:43:31
tags: [js]
categories: 前端学习
thumbnail: https://s3.ax1x.com/2021/02/09/ydmH0S.png
cover: https://s3.ax1x.com/2021/02/09/ydmH0S.png
---

主要内容：JavaScript、canvas、一张图片

<!-- more -->

# 代码

canvas 展示验证码，图片作为验证码的背景

准备 `<canvas>` 标签

```html
<canvas id="canvas" width="100" height="40"></canvas>
```

code 保存生成的随机数，这里生成长度为 4 的验证码

```js
let code = "";
const canvas = document.getElementById("canvas");
// 调用方法生成验证码
createCode(4);
// 点击刷新验证码
canvas.addEventListener("click", () => {
  createCode(4);
});
```

生成验证码的方法

```js
function createCode(length) {
  // console.log(createCode)
  code = "";
  //验证码的长度
  let codeLength = parseInt(length);
  //  生成的验证码内容
  let codeChars = new Array(
    2,
    3,
    4,
    5,
    6,
    7,
    8,
    9,
    "a",
    "b",
    "c",
    "d",
    "e",
    "f",
    "g",
    "h",
    "i",
    "j",
    "k",
    "l",
    "m",
    "n",
    "o",
    "p",
    "q",
    "r",
    "s",
    "t",
    "u",
    "v",
    "w",
    "x",
    "y",
    "z",
    "A",
    "B",
    "C",
    "D",
    "E",
    "F",
    "G",
    "H",
    "I",
    "J",
    "K",
    "L",
    "M",
    "N",
    "O",
    "P",
    "Q",
    "R",
    "S",
    "T",
    "U",
    "V",
    "W",
    "X",
    "Y",
    "Z"
  );
  //循环组成验证码的字符串
  for (let i = 0; i < codeLength; i++) {
    //获取随机验证码下标
    let charNum = Math.floor(Math.random() * 60);

    //组合成指定字符验证码
    code += codeChars[charNum];
  }
  if (code) {
    // var canvas = document.getElementById('canvas')
    let ctx = canvas.getContext("2d");
    let img = document.createElement("img");
    img.src = "./images/verify.png";
    img.onload = function () {
      ctx.drawImage(img, 0, 0, 90, 40);
      ctx.font = "20px Verdana";
      // 创建一个渐变
      let gradient = ctx.createLinearGradient(0, 0, canvas.width, 0);
      gradient.addColorStop("0", "darkorange");
      gradient.addColorStop("0.4", "blue");
      gradient.addColorStop("0.5", "darkgreen");
      gradient.addColorStop("0.6", "orange");
      gradient.addColorStop("0.7", "darkcyan");
      // 填充一个渐变
      ctx.fillStyle = gradient;
      ctx.fillText(code[0], 15, 20 + Math.floor(Math.random() * 6));
      ctx.fillText(code[1], 25, 20 + Math.floor(Math.random() * 6));
      ctx.fillText(code[2], 45, 20 + Math.floor(Math.random() * 6));
      ctx.fillText(code[3], 60, 20 + Math.floor(Math.random() * 6));
    };
  }
}
```

# 🎨🎨

## canvas.getContext

> 方法返回`canvas`的上下文，如果上下文没有定义则返回 `null`。（<a href="https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/getContext">MDN</a>）

canvas.getContext(contextType, contextAttributes)

<span style="color: darkcyan">contextType</span> （上下文类型）的参数有：

- `2d`：建立一个二维渲染上下文
- `webgl`：创建一个三维渲染上下文（<a href="https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API" style="color: darkred">某些版本的浏览器不兼容</a>）
- `bitmaprenderer`：接口表示能够被绘制到 canvas 上的位图图像

<span style="color: darkcyan">contextAttributes</span> （上下文属性）的参数有：

- **`alpha`**: [`boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/boolean)值表明 canvas 包含一个[`alpha`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/alpha)通道. 如果设置为 false, 浏览器将认为 canvas 背景总是不透明的, 这样可以加速绘制透明的内容和图片

## createLinearGradient(x0, y0, x1, y1)

> 方法创建一个沿参数坐标指定的直线的渐变。(<a href="https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/createLinearGradient">MDN</a>)

x0：起点的 x 轴坐标。

y0：起点的 y 轴坐标。

x1：终点的 x 轴坐标。

y1：终点的 y 轴坐标。

## addColorStop(offset, color)

> 添加一个由`偏移值`和`颜色值`指定的断点到渐变。（<a href="https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasGradient/addColorStop">MDN</a>）

`offset`：0~1 的值

`color`：颜色

<!-- <h1><b style="color: orange; font-size: 100px">验证<b><del style="color: pink">🗑码</del></h1>-->
