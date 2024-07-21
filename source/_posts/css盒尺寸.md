---
title: css盒尺寸
toc: true
date: 2021-02-18 12:24:02
tags: [css]
categories: 前端学习
thumbnail: https://s3.ax1x.com/2021/02/18/yRcsnx.png
cover: https://s3.ax1x.com/2021/02/18/yRcsnx.png
---

📃主要内容：content box、padding box、border box、margin box  ~~ 读书笔记

📖参考书本：《CSS世界》

<!-- more -->

> css的很多效果并不是一个属性就能搞定，往往是几个属性一起作用的结果。

# content

## content与替换元素

替换元素就是其内容是可以被替换的，如：`<imag src="xxx.jpg">`，替换元素的一些特性：

- 内容的外观不受页面上的css影响
- 有自己的尺寸。可以不用`style`属性，直接写`width`，`height`。默认尺寸为**300px x 150px**（不包括边框）
- 在很多css属性上有自己的表现规则

### 替换元素的默认display值

所有的替换元素都是内联水平，也就是替换元素和替换元素、替换元素和文字都是可以在一行内显示的。

`<input>` 和 `<button>` 的区别：两种按钮的默认 `white-space` 值不一样，input 是 `pre` ，button 是 `normal`。所以按钮文字多的时候，input 的内容不会换行，button会换行。

### 替换元素的尺寸计算规则

由内到外分为3类：固有尺寸、HTML尺寸和CSS尺寸。

1. 固有尺寸是替换元素本身的尺寸
2. HTML尺寸就是`<img>` 标签的 <span style="color: darkcyan">width</span>，<span style="color: darkorange">height</span>和 `<textare>` 的<span style="color: darkcyan">rows</span>，<span style="color: darkorange">cols</span>
3. CSS尺寸是指通过css的<span style="color: darkcyan">width</span>，<span style="color: darkorange">height</span>等设置的尺寸，对应盒尺寸中的**content box**

这三种尺寸的计算规则：

- 如果没有CSS尺寸和HTML尺寸，使用固有尺寸作为最终宽高

- 如果没有CSS尺寸，使用HTML尺寸

- 如果有CSS尺寸，最终由CSS属性决定，

  ```css
  img{ width: 200px; height: 150px; }
  ```

  ```html
  <img src="abc.jpg" width="150px" height="100px">
  ```

  最终的尺寸是 <i>200px x 200px</i>

- ⭐如果固有尺寸含有固定的*宽高比例*，同时<span style="color: red">仅仅</span>设置了宽度或者高度，则依然按照固有的宽高比例来显示

在 `<img>` 中，没有 **src** 和 **src=" "** 是不一样的，前者不会发送请求，后者在一些浏览器中会发送请求，请求当前页面数据。

- 在 Firefox 中，没有 src 的 img标签不是替换元素，而是内联元素。

在 CSS 中**替换元素内容的固有尺寸是无法改变的**。❓平常设置的 width 和 height 会对图片的尺寸进行影响，是因为图片中的 content 替换内容默认的适配方式是**填充**（fill），也就是外部设定的尺寸多大，图片就填满。使用`object-fill`可以修改这个默认值。

### 替换元素与非替换元素的差别

#### 差一个src

成为普通的内联元素，在 Firefox 中，不使用 src 属性；在 Chrome 中除了前面的条件，还要加上**属性值不为空**的`alt`。如：`<img alt="any">`。注：

1. 不能有 src 属性。
2. 不能使用 content 属性生成图片。（对于Chrome）
3. 需要有属性值不为空的 alt。（对于Chrome）
4. Firefox 中 `::before`的 content 值会被无视

💡实用例子：👉<a href="https://demo.cssworld.cn/4/1-2.php">CSS世界</a>

#### 差一个CSS content属性

从理论层面来讲，content属性决定了是替换元素还是非替换元素。一些用法：

1. 如果图片本身有 src 地址，可以使用 content 属性把图片的内容置换，可以用于hover的切换效果。
   - 这样使用只是视觉上的效果，使用右键保存图片的时候保存的还是**原来**src的图片
2. 使用 content，可以使得普通元素变为替换元素

### content与替换元素的关系

CSS世界中，把content属性生成的对象称为“匿名替换元素”。

content属性生成的内容和普通元素内容的差异：

1. 使用content生成的文本是无法选中、复制的。
2. 不要将重要的信息放在content，因为对SEO不友好

## content内容生成

1. 经常用在 `::before` 和 `::after` 中，用于添加一些描述、解释、绘制图案、小图标
2. content图片生成，使用 `content: url(1.jpg)` ，但这样不利于操作图片的尺寸，更多会使用：`content: '',background:url(1.jpg)`
3. content 计数器

### content计数器 ⭐

只是用css实现的计数器（随着元素数目增加，数值随着增加）

对应2个属性（counter-reset、counter-increment）和1个方法（ counter() / counters() ）

**counter-reset**：主要作用是给计数器起名，并且设定开始的数值（默认起始数字为0）

- 这个值可以是负数，在Chrome中还可以是小数。不过 2.99 会变成 2
- 可以命名多个计数器

**counter-increment**：计数器递增。值为 counter-reset 中设定好的计数器名称；后面可以指定数字，表示每次计算的变化值。

- 默认值每次增加1。因为要和 counter-reset 一起搭配使用，这也是为什么 counter-reset 的默认值是 0 的原因。
- 可以作用在伪元素上。
- 变化值也可以是负数，递减效果。

**counter() / counters()**

- 作用就是输出计数，里面参数是计数器的名字
- 第2个参数 *style*，支持的值就是 *list-style-type* 属性的那些值。递增的不一定是数字，可能是字母之类的等等
- 支持级联，也就是一个 content 属性可以有多个 counter() 方法

**counters()**

- 左右即是生成 1.1、1.2、1.3 这样的序号
- 通过子元素对父元素的 counter-rest、counters()实现嵌套，一个容器里的 counter-rest 是唯一的
- 显示 content 计数器的 DOM 元素一定要在 counter-increment 元素的**后面**。

```html
<div id="show"></div>
```

```css
 #show {
    counter-reset: sixsix 0 sevenseven 1;
    counter-increment: sixsix 2 sevenseven 3;
}
#show::before{
    content: counter(sixsix);
}
#show::after{
    content: counter(sixsix) '\A' counter(sevenseven);
}
```

- `'\A'` 表示换行

# padding

css中默认的 *box-sizing* 是 *content-box*，使用 *padding* 是会增加元素的尺寸的。

padding的作用不会影响布局效果，只是在视觉上加大尺寸；但是给父级元素加上`overflow: auto;`则会影响页面效果。

## padding的百分值

padding不支持负值；在百分比的计算中，水平方向和垂直方向的值都是相对于宽度计算的。

在某些场景中，直接使用 `padding: 50%;` 就可以实现一个正方形；`padding: 25% 50%;` 就可以实现一个2:1的矩形。

```html
<div class="box">
   <div class="item"></div>
 </div>
```

```css
.box {
    padding: 5% 10%;
    position: relative;
}
.item {
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
    background-color: darkcyan;
}
```

- 当然可以用作其他元素，例如固定比例的图片

**对于内联元素**，padding的区域是跟着内联盒子的，如果内联盒子换行了，那padding也会跟着变化。

内联元素的垂直padding会让“幽灵空白节点”显现

## padding绘制图形

padding与background-clip搭配实现简单的图形绘制

🔻三条杠菜单

```html
<i class="icon-menu"></i>
```

```css
.icon-menu {
    display: inline-block;
    width: 140px; height: 10px;
    padding: 35px 0;
    border-top: 10px solid;
    border-bottom: 10px solid;
    background-color: currentColor;
    background-clip: content-box;
}
```

🔻小圆点

```html
<i class="icon-dot"></i>
```

```css
.icon-dot {
    display: inline-block;
    width: 100px; height: 100px;
    padding: 10px;
    border: 10px solid;    
    border-radius: 50%;
    background-color: currentColor;
    background-clip: content-box;
}
```

# border

> border-width 不支持百分比，如果支持，那边框不就随着屏幕的变大而变大，这样不合理。outline、box-shadow、text-shadow也不支持

border的默认值时 none，使用 `border: 2px;`、`border: yellow` 都是无效果的，使用  `border: solid` 有效果，会出现一个 3px 的边框。

border-style 为 double 的时候，要到 3px 才会出现双线的效果。

border的其他风格：inset、outset、groove（沟槽）、ridge（山脊）

## border颜色

border-color 和 color：在没有设置 border-color 的时候，会以color的颜色为准。

## border其他使用

### 增加点击范围

```css
icon{
    width: 16px;
    height: 16px;
    border: 11px solid transparent;
}
```

### 画三角形

```css
div{
    width: 0;
    border: 10px solid;
    border-color: red transparent transparent transparent;
}

/* 或者*/
div{
    width: 0;
    border: solid;
    border-width: 10px 20px;
    border-color: red transparent transparent transparent;
}
```

- 按照顺时针的顺序

### 梯形

```css
div{
    width: 10px;
    height: 10px;
    border: 10px solid;
    border-color: red transparent transparent;
}
```



# margin

## margin合并

在块级元素中，margin-button 和 margin-top 会发生合并。（块级元素，垂直方向）**不包括**浮动和绝对定位的元素

两个元素之间有一个空的块级元素，也会合并。比如有个空的`<div>`

1. 相邻兄弟的margin合并，这是最常见的合并。如：

   ```html
   <p>qwe</p>
   <p>asd</p>
   ```

   ```css
   p { margin: 1em }
   ```

   

   ![兄弟margin合并](https://s3.ax1x.com/2021/02/20/y54L59.png)

2. 父级和第1个或者最后一个子元素

   ```html
   <div class="father">
     <div class="son"></div>
   </div>
   ```

   ```css
   .father {
       width: 200px;
       height: 200px;
       background-color: aqua;
       margin-top: 50px;
   }
     .son {
       margin-top: 50px;
     }
   ```

   ![父子margin合并](https://s3.ax1x.com/2021/02/20/y559bD.png)

### 阻止合并

   **对于margin-top**（有一个满足即可）

   1. 父元素设置为<a href="#" style="color: gold">块状格式上下文元素</a>
   2. 父元素设置border-top值
   3. 父元素设置padding-top值
   4. 父元素和子元素之间添加一个内联元素进行隔离

   **对于margin-bottom**

   1. 和上面4点一样，但是属性取反
   2. 父元素设置 height、min-height 或 max-height

   使用 `overflow: hidden;` 将父级元素设置为块状格式上下文。

   > jQuery中使用 `$().slideUp` 和 `$().slideDown` 方法时，动画有卡顿现象，可能就是这个合并造成的。

### margin合并的计算规则

- 正值和正值取最大值
- 正值和负值，相加
- 两个负值取最小的

### margin合并的意义

不用担心列表之间的间隙会太大，如果有空元素，也会合并起来，避免页面布局混乱

## margin:auto

作用机制：

1. 如果一侧是定制，一侧是`auto`，那 auto 就是剩下的空间
2. 如果两侧都是 auto，那就是平分空间

如果想让某个块状元素右对齐，除了使用 *float：right;*，还可以使用 `margin-left: auto;`

- 因为margin的默认大小是 0，然后 margin-left：auto 就百分百填满了

❓ 为什么确定高度，使用 *margin: auto;* 却无法垂直居中

因为 margin: auto 计算有一个前提条件，就是 width 或者 height 为 auto 的时候，元素是具有对应方向的自动填充的特性。

**方法1**

父元素使用 `writing-mode: vertical-lr`，垂直方向就可以居中了，但是水平方向就没了。、

**方法2**

```css
      .father {
        height: 150px; width: 150px;
        background-color: antiquewhite;
        position: relative;
      }
      .son {
        height: 50px; width: 50px;
        margin: auto;
        background-color: aqua;
        position: absolute;
        top: 0; right: 0; bottom: 0; left: 0;
      }
```

- 其实方法还有很多，以后做个总结❓

## margin无效情况

1. display 为 inline 的*非替换元素* 的**垂直**margin是无效的。对于替换元素，如img就没有这种烦恼

2. 表格中 display 是 table-cell 或者 table-row 的时候

3. margin合并的时候没效果，但是值超过合并的值或者是负值就会有效果

4. 绝对定位元素非定位方向的margin。比如：

   1. ```css
      img{
          top: 50%;
          left: 50%;
          margin-right: 25px;
      }
      ```

      - 实则上是有效果，但是没表现出来。因为绝对定位是独立渲染的，自己玩，影响不了兄弟元素。

5. 定高元素的子元素的 margin-bottom 或者 宽度定死的 margin-right

6. 使用 float 的时候，比如图片。

   1. ```html
      <div class="box">
        <img src="./img/chigua.jpg" alt="" />
        <p>asd</p>
      </div>
      ```

      ```css
      .box > img {
          float: left;
          width: 233px;
        }
        .box > p {
          margin-left: 200px;
        }
      ```

      要想达到margin-left有效，那margin-left的值要**大于** 233px 才行，这是 float 作用效果。

      如果文字足够多的情况下，设置大于 233px

      ![margin-left有效果](https://s3.ax1x.com/2021/02/20/y55mKf.png)

      

   如果不要margin-left：

   ![不要margin-left](https://s3.ax1x.com/2021/02/20/y55G2q.png)

> 这里的文字是环绕的。



⏳持续补充....