---
title: jQuery基本内容
toc: true
date: 2021-03-09 13:27:31
tags: [js]
categories: 前端学习
thumbnail: https://s3.ax1x.com/2021/03/14/60dKb9.png
cover: https://s3.ax1x.com/2021/03/14/60dKb9.png
---

<a href="https://jquery.com/download/"><span style="color: darkcyan;">jQuery</span>官方下载</a>

<!-- more -->

<span style="color: gold;">write less, do more</span>

jQuery 是 JavaScript 框架库，通过 JavaScript 函数封装来简化 HTML 和 JavaScript 之间的操作。

跨浏览器，支持 `IE6~IE11、Firefox、Chrome，支持CSS1~CSS3`

# jQuery 对象

jQuery 对象是对 DOM 的一层包装

![jquery对象](https://s3.ax1x.com/2021/03/09/63nClt.png)

1. $(document)：表示将 document 对象转化为 jQuery 对象。
2. 下标为`0`的元素表示内部的 DOM 对象，就是 document 对象
3. `length` 表示其内部 DOM 对象的个数，一个 jQuery 对象可以有**多个**DOM 对象
4. `__protp__`可以查看这鬼对象的原形（jQuery 本身）所具有的属性和方法

# 元素操作

## jQuery 选择器

|  基本选择器  |  层级选择器   |   基本过滤选择器   |       内容选择器        |  可见性选择器   | 属性选择器 |   子元素选择器    |
| :----------: | :-----------: | :----------------: | :---------------------: | :-------------: | :--------: | :---------------: |
| 标签、id、类 | 空格、>、+、~ | :first、:odd、等等 | :contains、:empty、等等 | :show、:visible | value 等等 | :first-child 等等 |

### 基本选择器

|        选择器         |               描述               |       例子        |
| :-------------------: | :------------------------------: | :---------------: |
|        element        |        匹配元素名所有元素        |      $('a')       |
|          #id          |      匹配 id，**一个元素**       |     $('#btn')     |
|        .class         |        匹配类名的所欲元素        |  $('.btn-style')  |
| selector1, selecttor2 | 匹配多个（一个引号内用逗号隔开） | $('a, span, div') |

可组合使用

### 层级选择器

|       选择器       |             描述             |                           例子                           |
| :----------------: | :--------------------------: | :------------------------------------------------------: |
| selector selector1 |  选择祖先元素下的所有子元素  |                      $('div .test')                      |
|   parent > child   |     父元素下的所有子元素     |                   $('.show > .color')                    |
|     pre + next     | 当前元素相邻的下一个同级元素 | $('div + .show')，获取 div 下一个 class 名为`show`的元素 |
|   pre ~ siblings   | 当前元素**后**的所有同级元素 |                      $('.show ~ a')                      |

### 基本过滤选择器

|     选择器     |                 描述                  |                                    例子                                     |
| :------------: | :-----------------------------------: | :-------------------------------------------------------------------------: |
|     :first     |        指定选择器的第一个元素         |                               $('li :first')                                |
|     :last      |             最后一个元素              |                                $('li :last)                                 |
|     :even      | 指定选择器索引值为偶数的行，从 0 开始 |                                $('li :even')                                |
|      :odd      |                 奇数                  |                                $('li :odd')                                 |
|   :eq(index)   | 获取索引等于 index 的元素，从 0 开始  |                                $('li:eq(2)')                                |
|   :gt(index)   |        索引值大于 index 的元素        |                                $('li:gt(2)')                                |
|   :lt(index)   |           小于 index 的元素           |                                $('li:lt(2)')                                |
| :not(selector) |        除指定选择器之外的元素         |                            $('li:not(li:eq(2))')                            |
|     :focus     |           获取当前焦点元素            |                              $('input:focus')                               |
|   :animated    |        正在执行动画效果的元素         |                           $('li:not(:animated)')                            |
|    :target     |  由文档 URI 的格式化识别码表示的元素  | 若 URI 为`http://hello/#hi`，则 $('div:target')会获得 `<div id="hi"></div>` |

### 内容选择器

根据元素的内容获得指定元素

|     选择器      |           描述           |           例子            |
| :-------------: | :----------------------: | :-----------------------: |
| :contains(text) | 内容包含 text 文本的元素 | $('li:contains("hello")') |
|     :empty      |      内容为空的元素      |       $('li:empty')       |
| :has(selector)  |  内容有指定选择器的元素  |     $('li:has("a")')      |
|     :parent     | 内容不为空的元素（特殊） |      $('li:patent')       |

### 可见性选择器

|  选择器  |       描述       |                 例子                 |
| :------: | :--------------: | :----------------------------------: |
| :hidden  | 获取所有隐藏元素 | $('li:hidden')，获取所有隐藏的`<li>` |
| :visible |   所有可见元素   |           $('li:visible')            |

- 当 `display: none;`的时候，可以通过 `:hidden` 来获取隐藏的元素，反之亦然。

### 属性选择器

根据元素的属性获取指定元素。例如：获取 _class_ 值为 _show_ 的 `<div>`

|          选择器           |                    描述                    |                                           实例                                            |
| :-----------------------: | :----------------------------------------: | :---------------------------------------------------------------------------------------: |
|          [attr]           |               指定属性的元素               |                        $('div[class]')，获取含有 class 的所有 div                         |
|       [attr=value]        |           属性值为 value 的元素            |                                   $('div[class=show]')                                    |
|       [attr^=value]       |         属性值以 value 开始的元素          |                                   $('div[class^=btn]')                                    |
|       [attr$=value]       |            以 value 结尾的元素             |                                  $('div[class$=active]')                                  |
|       [attr*=value]       |          属性值包含 value 的元素           |            $('div[class*="-"]')，`-`为<span style="color: red;">字符串</span>             |
|       [attr~=value]       | 获取元素的属性值包含一个 value，以空格分开 | $('div[class~="box"]')，获取 class 中有 box 或者通过空格分开且包含 box 的元素，如`a box`  |
| `[attr1][attr]...[attrN]` |         获取同时拥有多个属性的元素         | `$('input[id][name$='usr]')`，获取同时含有 _id_ 和属性值以 usr 结尾的 _name_ 属性的 input |

### 子元素选择器

|                 选择器                 |                              描述                               |
| :------------------------------------: | :-------------------------------------------------------------: |
|    :nth-child(index/even/odd/公式)     |                       index 默认以 1 开始                       |
|              :first-child              |                          第一个子元素                           |
|              :last-child               |                            最后一个                             |
|              :only-child               |                如果当前元素是唯一的子元素，匹配                 |
|  :nth-last-child(index/even/odd/公式)  |  所有它们父元素的第 n 个子元素，计数从最后一个元素开始到第一个  |
|   :nth-of-type(index/even/odd/公式)    | 选择同属于一个父元素之下，并且标签名相同的子元素中的第 n 个元素 |
|             :first-of-type             |                 所有相同元素名称的第一个子元素                  |
|             :last-of-type              |                            最后一个                             |
|             :only-of-type              |           选择所欲没有兄弟元素，且具有相同名称的元素            |
| :nth-last-of-type(index/even/odd/公式) |  选择所欲它们的父级元素的第 n 个子元素，从最后一个元素开始计数  |

### 表单元选择器

|  选择器   |                       描述                       |
| :-------: | :----------------------------------------------: |
|  :input   | 获取页面所有表单元素，包括`<select>、<textatea>` |
|   :text   |                 页面中所有文本框                 |
| :password |                    所有密码框                    |
|  :radio   |                   所有单选按钮                   |
| :checkbox |                   所有复选按钮                   |
|  :submit  |                 获取 submit 按钮                 |
|  :reset   |                    reset 按钮                    |
|  :image   |            获取`type='image'`的图像域            |
|  :button  |        button 按钮，包括 `type='button'`         |
|   :file   |              `type='file'`的文件域               |
|  :hidden  |                  获取隐藏表单相                  |
| :enabled  |                获取所有可用表单项                |
| :disabled |                  获取不可用...                   |
| :checked  |       获取所有选中，针对 radio 和 checkbox       |
| :selected |                   针对 select                    |

> <span style="color: darkorange;">注意</span> `$('input')` 和 `$(':input')`不一样，前者获取的是 `<input>`，后者获得所有表单控件

## 元素遍历

**each( function( index, element ) {} )**

- index：当前元素的索引位置，从 0 开始
- element：当前元素

这方法的参数是一个回调函数，每个匹配的元素都会执行这个函数

```html
<ul>
  <li>HTML</li>
  <li>CSS</li>
  <li>JavaScript</li>
</ul>
```

```js
$("li").each((index, element) => {
  console.log(index);
  console.log(element);
  console.log(element.text());
});
```

![遍历](https://s3.ax1x.com/2021/03/09/68Fuiq.png)

## 元素内容

|     方法      |                       描述                       |
| :-----------: | :----------------------------------------------: |
|    html()     |        **获取**第一个匹配元素的 HTML 内容        |
| html(content) |        **设置**第一个匹配元素的 HTML 内容        |
|    text()     | **获取**所有匹配元素包含的文本内容组合起来的文本 |
| text(content) |          **设置**所有匹配元素的文本内容          |
|     val()     |          **获取**表单元素的 _value_ 值           |
|  val(value)   |          **设置**表单元素的 _value_ 值           |

`val()`还可以操作`select、radio、checkbox` 的选中情况，当获取的元素是 `<select>`的时候，返回的结果是*一个包含所选值的数组*；要为表单元素设置选中情况是，可以传递数组参数。

## 元素样式

|        方法        |                       描述                       |
| :----------------: | :----------------------------------------------: |
|     css(name)      |             获取第一个匹配元素的样式             |
|  css(properties)   |           以*键值* 的形式设置匹配样式            |
|  css(name, value)  |              `css('color', 'cyan')`              |
|      width()       |       获取第一个匹配元素的当前宽度（数值）       |
|    width(value)    |      设置宽度（可以是字符串，也可以是数值）      |
|      height()      |                   获取高度...                    |
|   height(value)    |                   设置高度...                    |
|      offset()      | 获取元素位置，返回一个对象，包含 _left_ 和 _top_ |
| offset(properties) | 利用对象设置元素的位置，必须包含 _left_ 和 _top_ |

`css()`中，`-` 要去掉，然后属性换成大写，例如：`background-color` 改为 `backgroundColor`。

`css()`里面传对象：

```js
ele.css({ color: "red", backgroundColor: "green" });
```

## 元素筛选

### 查找

|       方法       |                           描述                           |
| :--------------: | :------------------------------------------------------: |
|    find(expr)    |              搜索所有与指定表达式匹配的元素              |
| parents([expr])  | 取得一个包含所有匹配元素的祖先元素的集合（不包含根元素） |
| parents([expr])  |     取得一个包含所有匹配元素的唯一父级元素的元素集合     |
| siblings([expr]) |               取得所有同级元素（不分上下）               |
|   next([expr])   |              匹配紧邻的同级元素的下一个元素              |
|   pre([expr])    |              匹配紧邻的同级元素的上一个元素              |

### 过滤

|            方法            |                                 描述                                  |
| :------------------------: | :-------------------------------------------------------------------: |
|         eq(index)          |                            获取第 N 个元素                            |
| filter(expr\|obj\|ele\|fn) |            使用选择器、对象、元素或函数完成指定元素的筛选             |
|      hasClass(class)       |            检查是否含有某个 _class_，返回 `true`或`false`             |
|          is(expr)          | 用一个表达式来检查当前选择的元素集合，若果其中有一个符合，返回 `true` |
|         has(expr)          |         保留包含特定后代的元素，去掉那些不含有指定后代的元素          |

## 元素属性

### 基本属性操作

|         方法         |                      描述                      |
| :------------------: | :--------------------------------------------: |
|      attr(name)      | 取得第一个匹配元素的属性值，否则返回 undefined |
|   attr(properties)   | 将一个键值对形式的对象设置为所有撇皮元素的属性 |
|  attr(name, value)   |          将所匹配的元素设置一个属性值          |
| attr(name, function) | 将函数的返回值作为所匹配元素的 _name_ 的属性值 |
|      prop(name)      | 取得第一个匹配元素的属性值，否则返回 undefined |
|   prop(properties)   | 将一个键值对形式的对象设置为所有撇皮元素的属性 |
|  prop(name, value)   |          将所匹配的元素设置一个属性值          |
| prop(name, function) | 将函数的返回值作为所匹配元素的 _name_ 的属性值 |
|   removeAttr(name)   |          从每个匹配元素中删除一个属性          |

`attr()`和`prop()`一样，不过在使用 `checked` 和 `selected`和`disabled`的时候使用 `prop()`。

这两个方法都只能获取一个匹配的属性，所以要配合`each()`来获取所有的属性

### class 属性操作

要操作元素的 _class_ 以达到动态样式，可以使用 `attr()`，但是只能操作一个，不够灵活。用一下方法：

|     方法      |   作用   |                       描述                       |
| :-----------: | :------: | :----------------------------------------------: |
|  addClass()   | 追加样式 |            为匹配的元素追加指定的类型            |
| removeClass() | 移除样式 |             全部删除或者删除指定类名             |
| toggleClass() | 切换样式 | 判断指定样式类是否存在，存在就删除，不存在就添加 |
|  hasClass()   | 判断样式 |                 判断是否有 class                 |

`addClass()`和`removeClass()`要添加多个样式的时候，中间用**空格**分开

# DOM 节点操作

## 节点追加

父子节点：在匹配到的元素内部添加指定的 _content_ 内容。

兄弟节点：在匹配到的元素外部添加指定的 _content_ 内容。

### 父子节点

|       方法        |                  描述                   |
| :---------------: | :-------------------------------------: |
|  append(content)  | 把 content 内容追加到匹配的元素内容尾部 |
| prepend(content)  | 把 content 内容追加到匹配的元素内容头部 |
| appendTo(content) | 把匹配到的内容插入到 content 内容的尾部 |
|     prependTo     | 把匹配到的内容插入到 content 内容的头部 |

### 兄弟节点

|         方法          |                    描述                     |
| :-------------------: | :-----------------------------------------: |
|    after(content)     |       把 content 内容插入到元素的尾部       |
|    before(content)    |       把 content 内容插入到元素的头部       |
| insertAfter(content)  | 把所有匹配到的内容插入到 content 元素的尾部 |
| insertBefore(content) | 把所有匹配到的内容插入到 content 元素的头部 |

**例子**

```html
<div class="list">
  <ul>
    <li>one</li>
    <li>two</li>
    <li>three</li>
  </ul>
</div>
<div class="add">+添加</div>
```

```js
$(".add").click(function () {
  var str = "<li>four</li>";
  $(".list li:lase-child").after(str);
});
```

## 节点替换

|         方法         |                     描述                      |
| :------------------: | :-------------------------------------------: |
| replaceWith(content) | 将所有匹配的元素替换成指定的 HTML 或 DOM 元素 |
| replaceAll(content)  | 用匹配的元素替换掉所有 selector 匹配到的元素  |

replaceWith()的参数是一个 _函数_ 的时候，它的返回值必须是字符串类型，用于完成指定的元素替换

## 节点删除

|      方法      |                             描述                              |
| :------------: | :-----------------------------------------------------------: |
|    empty()     |                清空元素内容，但不删除元素本身                 |
| remove([expr]) |  清空元素内容，并删除元素本身（可选参数 expr 用于筛选元素）   |
|    detach()    | 从 DOM 中删除所有匹配的元素（保留所有绑定事件、附加的数据等） |

**例**

```html
<ul>
  <li>吃饭</li>
  <li>睡觉</li>
  <li>打游戏</li>
</ul>
```

![原本](https://s3.ax1x.com/2021/03/11/6NDIjH.png)

```js
$("li:first-child").empty();
```

![使用empty](https://s3.ax1x.com/2021/03/11/6NDjC8.png)

```js
$("li:first-child").remove();
```

![使用remove](https://s3.ax1x.com/2021/03/11/6NrFU0.png)

## 节点复制

|      方法      |                          描述                           |
| :------------: | :-----------------------------------------------------: |
| clone([false]) | 复制匹配的元素并且选中这些复制的副本，默认参数为`false` |
|  clone(true)   |                 复制元素的所有事件处理                  |

# 事件操作

## 常用事件

**表单事件**

|              方法               |                描述                |
| :-----------------------------: | :--------------------------------: |
|   blur( [ [data], function ])   |         元素失去焦点时触发         |
|  focus( [ [data], function ])   |         元素获得焦点时触发         |
|  change( [ [data], function ])  |       元素的值发生改变时触发       |
| focusin( [ [data], function ])  | 在父元素上检测子元素获取焦点的情况 |
| focusout( [ [data], function ]) | 在父元素上检测子元素失去焦点的情况 |
|  select( [ [data], function ])  |    当文本框中的文本被选中时触发    |
|  submit( [ [data], function ])  |          当表单提交时触发          |

**键盘事件**

|               方法               |                    描述                     |
| :------------------------------: | :-----------------------------------------: |
| keydown( [ [data], function ] )  |             键盘按键按下时触发              |
| keypress( [ [data], function ] ) | 除 _Shift、Ctrl、Fn_ 等键盘，按键按下时触发 |
|  keyup( [ [data], function ] )   |             键盘按键弹起时触发              |

**鼠标事件**

|               方法                |              描述              |
| :-------------------------------: | :----------------------------: |
| mouseover( [ [data], function ] ) |         鼠标移入时触发         |
| mouseout( [ [data], function ] )  |         鼠标离开时触发         |
|   click( [ [data], function ] )   |         单击元素时触发         |
|  dbclick( [ [data], function ] )  |         双击元素时触发         |
| mousedown( [ [data], function ] ) | 鼠标在目标上方，并且按下时触发 |
|  mouseup( [ [data], function ] )  |     鼠标在目标上松开时触发     |

**浏览器事件**

|              方法              |             描述             |
| :----------------------------: | :--------------------------: |
| scroll( [ [data], function ] ) |     滚动条发生变化时触发     |
| resize( [ [data], function ] ) | 调整浏览器窗口大小的时候触发 |

**例 1：输入提示**

```html
<body>
  <p>用户：<input type="text" /></p>
  <p>密码：<input type="text" /></p>
  <p>邮箱：<input type="text" /></p>
</body>
<script>
  $("input[type=text]").focus(function () {
    let tips = $("<span></span>");
    tips.html("&nbsp;请按照要求输入");
    $("input:focus").after(tips);
  });
  $("input[type=text]").blur(function () {
    $(this).next().remove();
  });
</script>
```

**例 2：移动元素**

```html
<style>
  .box {
    width: 50px;
    height: 50px;
    background-color: darkcyan;
  }
</style>
<body>
  <div class="box"></div>
</body>
<script>
  $(document).keydown((e) => {
    let option = e.which;
    let box = $(".box");
    let top = box.offset().top;
    let left = box.offset().left;
    switch (option) {
      case 37:
        box.offset({ left: left - 10, top: top });
        break;
      case 38:
        box.offset({ left: left, top: top - 10 });
        break;
      case 39:
        box.offset({ left: left + 10, top: top });
        break;
      case 40:
        box.offset({ left: left, top: top + 10 });
        break;
    }
  });
</script>
```

## 页面加载操作

|   选项   |                          window.onload                           |      $(document).ready()      |
| :------: | :--------------------------------------------------------------: | :---------------------------: |
| 执行时机 | 必须等网页中的所有内容加载完成后（包括图片等之类的东西）才能执行 | 网页内的所有 DOM 结构加载完成 |
| 编写个数 |                         不能同时编写多个                         |       可以同时编写多个        |
| 简化写法 |                               没有                               |              $()              |

jQuery 的页面加载有 3 中写法：

```js
$(document).ready(function () {});

$().ready(function () {});

$(function () {});
```

## 事件绑定与切换

|                   方法                   |                   描述                    |
| :--------------------------------------: | :---------------------------------------: |
| on(events, [selector], [data], function) |    在元素上绑定一个或多个事件处理函数     |
|    off(events, [selector], function)     |    在元素上移除一个或多个事件处理函数     |
|      one(events, [data], function)       |  为每个匹配元素的事件绑定一次性处理函数   |
|          trigger(type, [data])           |       在每个匹配元素上触发某类事件        |
|       triggerHandler(type, [data])       | 同`trigger()`，但浏览器默认动作不会被触发 |
|            hover([over,]out)             |          元素移入和移出事件切换           |

- data：表示将要传递给事件处理函数的数据
- type：表示为元素添加的事件类型（多个用空格隔开）
- over 和 out 分别表示移入和移出时的事件处理函数

# 动画特效

jQuery 中有两种增加动画的方法：一种是内置的动画方法；另一种是通过 `animation()`进行自定义动画

## 常用动画

**基本**

|              方法               |        描述        |
| :-----------------------------: | :----------------: |
|  show([speed, [easing], [fn]])  | 显示隐藏的匹配元素 |
|  hide([speed, [easing], [fn]])  | 隐藏显示的匹配元素 |
| toggle([speed, [easing], [fn]]) | 元素显示与隐藏切换 |

**滑动**

|                 方法                 |               描述               |
| :----------------------------------: | :------------------------------: |
|  slideDown([speed, [easing], [fn]])  | 垂直滑动显示匹配元素（向下增大） |
|   slideUp([speed, [easing], [fn]])   | 垂直滑动显示匹配元素（向上减小） |
| slideToggle([speed, [easing], [fn]]) | 在 slideDown 和 slideUp 之间切换 |

**淡入淡出**

|                   方法                   |                    描述                    |
| :--------------------------------------: | :----------------------------------------: |
|     fadeIn([speed, [easing], [fn]])      |              淡入显示匹配元素              |
|     fadeOut([speed, [easing], [fn]])     |              淡出隐藏匹配元素              |
| fadeTo([speed, opacity, [easing], [fn]]) | 以淡入淡出方式将匹配元素调整到指定的透明度 |
|   fadeToggle([speed, [easing], [fn]])    |   在 fadeIn()和 fadeOut()两种效果间切换    |

- speed：动画的速度，单位是毫秒，也可以是 _slow，fast，normal_
- easing：切换效果，默认为 _swing_，还可以使用 _linear_
- opacity：透明度，_0~1_

```html
<style>
  .box div {
    width: 50px;
    height: 50px;
  }
  .red {
    background-color: red;
  }
  .green {
    background-color: green;
  }
  .yellow {
    background-color: yellow;
  }
  .orange {
    background-color: orange;
  }
</style>
<body>
  <div class="box">
    <div class="red"></div>
    <div class="green"></div>
    <div class="yellow"></div>
    <div class="orange"></div>
  </div>
</body>
<script>
  $(".box div").fadeTo(2000, 0.2);
  $(".box div").hover(
    function () {
      $(this).fadeTo(1, 1);
    },
    function () {
      $(this).fadeTo(1, 0.2);
    }
  );
</script>
```

## 自定义动画

|                  方法                   |                                 描述                                 |
| :-------------------------------------: | :------------------------------------------------------------------: |
| animate(params,[speed], [easing], [fn]) |                       用于创建自定义动画的函数                       |
|      $.speed([speed], [settings])       |             创建一个包含一组属性的对象用来定义自定义动画             |
|           queue([queueName])            |                    显示被选元素上要执行的函数列队                    |
|        delay(speed, [queueName])        |                设置一个延时来推迟执行队列中之后的项目                |
|        clear(speed, [queueName])        |                    从尚未运行的队列中移除所有项目                    |
|          dequeue([queueName])           |                  从队列移除下一个函数，然后执行函数                  |
|           finish([queueName])           | 停止当前正在运行的动画，删除所欲排队的动画，并完成匹配元素所有的动画 |
|     stop([clearQueue], [jumpToEnd])     |                  停止所欲在指定元素上正在运行的动画                  |

- params：表示一组包含动画最终属性值的集合
- settings：是 _easing_ 与 _fn_ 组成的一个对象集合
- queueName：表示列队名称，默认值为 _fx_（标准效果队列）
- clearQueue, jumpToEnd：都是布尔值，默认为 _false_。前者规定是否停止被选元素所有加入队列的动画；后者规定是否立即完成当前的动画。

**例 1**，简单的自定义动画

```html
<body>
  <input type="submit" id="btn" value="开始动画" />
  <div></div>
</body>
<script>
  $("#btn").click(() => {
    $("div").css({ background: "red", width: 0, height: 0 });
    let params = { width: "100px", height: "100px" };
    let settings = $.speed(2000, "linear");
    $("div").animate(params, settings);
  });
</script>
```

**例 2**，动画队列

```html
<style>
  div {
    position: absolute;
    background: red;
    width: 50px;
    height: 50px;
    display: none;
  }
</style>
<body>
  <p>队列长度为：<span></span></p>
  <div></div>
</body>
<script>
  const div = $("div");
  runQue();
  showQue();
  function runQue() {
    div
      .show("slow")
      .animate({ left: "+=200" }, 2000)
      .animate({ left: "-=200" }, 1500)
      .slideUp("normal", runQue)
      .queue(function () {
        $(this).css("background", "green").dequeue();
      });
  }
  function showQue() {
    $("span").text(div.queue("fx").length);
    setTimeout(showQue, 100);
  }
</script>
```

# jQuery 操作 Ajax

## 常用方法

**高级应用**

|               方法                |                         描述                         |
| :-------------------------------: | :--------------------------------------------------: |
| $.get(url, [data], [fn], [type])  |            通过远程 HTTP GET 请求载入信息            |
| $.post(url, [data], [fn], [type]) |           通过远程 HTTP POST 请求载入信息            |
|   $.getJSON(url, [data], [fn])    |         通过远程 HTTP GET 请求载入 JSON 数据         |
|      $.getScript(url, [fn])       | 通过远程 HTTP GET 请求载入并执行一个 JavaScript 文件 |
| 元素对象.load(url, [data], [fn])  |        载入远程 HTML 文件代码并插入至 DOM 中         |

- fn：请求成功时执行的回调函数
- type：设置服务器返回的数据类型，如 HTML、JSON、TEXT 等

**底层应用**

|          方法          |            描述            |
| :--------------------: | :------------------------: |
| $.ajax(url, [options]) | 通过 HTTP 请求加载远程数据 |
|  $.ajaxSetup(options)  |   设置全局 Ajax 默认选项   |

- options：设置 Ajax 请求的相关选项

options 选项：

|    选项     |                               描述                               |
| :---------: | :--------------------------------------------------------------: |
|     url     |                    处理 Ajax 请求的服务器地址                    |
|    data     |               发送 Ajax 请求时传递的参数（string）               |
|   success   |                  Ajax 请求成功时触发的回调函数                   |
|    type     |                 发送 HTTP 请求方式，如 get、post                 |
|  datatype   |               期待的返回值类型，如：html、json 等                |
|    async    |                     是否异步，默认值是 true                      |
|    cache    |                     是否缓存，默认值是 true                      |
| contentType | 请求头，默认为 `application/x-www-form-urlencoded;charset=UTF-8` |
|  complete   |         服务器接收完 Ajax 请求传递的数据后触发的回调函数         |
|    jsonp    |              在一个 jsonp 请求中重写回调函数的名称               |

**例 1：**$.post()

```js
$.post(
  "index.php",
  { id: 2, name: "zs" },
  function (msg) {
    console.log(msg.id + "-" + msg.name);
  },
  "json"
);
```

**例 2：**$.ajax()

> 在这个方法汇总，通过设置 _options_ 可以实现 `$.get()， $.post()，$.getJSON()，$.getScript()` 一样的效果。

只发送 GET 请求

```js
$.ajax("index.php");
```

发送 GET 请求并传递数据，接收返回结果

```js
$.ajax("index.php", {
  data: { book: "php", price: 50 },
  success: function (msg) {
    console.log(msg);
  },
});
```

只配置 settings 参数，同样实现 ajax 操作

```js
$.ajax({
  type: "GET",
  url: "index.php",
  data: { id: 2, name: "js" },
  success: function (msg) {
    console.log(msg);
  },
});
```

使用 `ajaxSetup()`预先设置全局 Ajax 请求的参数，实现全局共享，避免频发交互时发生错误

```js
// 预先设置全局参数
$.ajaxSetup({
  type: "GET",
  url: "index.php",
  data: { id: 2, name: "js" },
  success: function (msg) {
    console.log(msg);
  },
});

// 执行 ajax 操作
$.ajax();
```

- 预设之后，再调用`$.ajax()、$.get()、$.post()`就会方便许多

## 其他相关方法

**辅助函数**

|       方法       |                    描述                    |
| :--------------: | :----------------------------------------: |
|   $.param(obj)   |         创建数组或对象的序列化表示         |
|   serialize()    | 通过序列化表单值，创建 URL 编码文本字符串  |
| serializeArray() | 通过序列化表单值，创建对象数组（名称和值） |

在 ajax 操作时，如果要将对象保存的数据、表单提交的数据转换为 URL 参数字符串，为了传递的参数中含有特殊字符，可以使用辅助函数

**例 1：**序列化对象

```js
let data = { id: 2, name: "Jerry", skill: ["html", "css", "js"] };
let seri_data = $.param(data);
let deseri_data = decodeURIComponent(seri_data);
console.log(seri_data);
console.log(deseri_data);
```

- decodeURIComponent()：JavaScript 中用于 URI 解码的函数

**例 2：**序列化表单数据

```html
<form>
  <p>姓名：<input type="text" name="username" /></p>
  <p>
    爱好：<input type="checkbox" name="hobby[]" value="swiming" />游泳
    <input type="checkbox" name="hobby[]" value="reading" />读书
    <input type="checkbox" name="hobby[]" value="running" />跑步
  </p>
  <p>描述：<textarea name="desc" cols="40" rows="5"></textarea></p>
  <input type="button" value="提交" />
</form>
<script src="jquery-1.12.4.min.js"></script>
<script>
  $("input[type=button]").on("click", function () {
    console.log($("form").serialize());
    console.log($("form").serializeArray());
  });
</script>
```

![序列化表单数据](https://s3.ax1x.com/2021/03/14/60nuFA.png)

**Ajax 事件**

|       方法       |                 函数                 |
| :--------------: | :----------------------------------: |
| ajaxComplete(fn) |     请求完成时触发的事件执行函数     |
|  ajaxError(fn)   |     请求错误时触发的事件执行函数     |
|   ajaxSend(fn)   |   请求发送**前**触发的事件执行函数   |
|  ajaxStart(fn)   | 请求发送**开始时**触发的事件执行函数 |
|   ajaxStop(fn)   | 请求发送**结束时**触发的事件执行函数 |
| ajaxSuccess(fn)  | 请求发送**成功时**触发的事件执行函数 |

- 事件发生的顺序 `start > send > success >error > compelete`

```js
$(document).ajaxError(function () {
  console.log("ajaxError");
});
$.post(
  "index.php",
  { id: 2, name: "JS" },
  function (msg) {
    console.log(msg.id + "-" + msg.name);
  },
  "xml"
);
```

# 元素操作案例

## 折叠菜单

样式

```html
<style>
  ul {
    list-style: none;
    padding: 0;
    margin: 0;
  }
  div {
    width: 150px;
    border: 1px solid #515e7b;
    margin: 10px;
  }
  div li {
    background: #515e7b;
    border-bottom: 1px solid #fff;
  }
  div li a {
    text-decoration: none;
    color: #fff;
    font-size: 16px;
    height: 40px;
    line-height: 40px;
    padding-left: 10px;
  }
  div li a:hover {
    text-decoration: underline;
  }
  .wrap {
    width: 150px;
    display: none;
  }
  .wrap li {
    background: #fff;
    margin: 0;
  }
  .wrap li a {
    color: #3b475f;
    font-size: 12px;
  }
</style>
```

DOM

```html
<div id="fold">
  <ul>
    <li>
      <a href="#">信息管理</a>
      <ul class="wrap">
        <li><a href="#">未读信息</a></li>
        <li><a href="#">已读信息</a></li>
        <li><a href="#">信息列表</a></li>
      </ul>
    </li>
    <li>
      <a href="#">商品管理</a>
      <ul class="wrap">
        <li><a href="#">商品添加</a></li>
        <li><a href="#">商品列表</a></li>
        <li><a href="#">商品分类</a></li>
      </ul>
    </li>
    <li>
      <a href="#">用户管理</a>
      <ul class="wrap">
        <li><a href="#">权限设置</a></li>
        <li><a href="#">用户列表</a></li>
        <li><a href="#">重置密码</a></li>
      </ul>
    </li>
  </ul>
</div>
```

JavaScript jQuery

```js
// 默认情况下，显示第一个分类下的菜单
$("#fold>ul>li:first").find(".wrap").css({ display: "block" });
// 根据用户单击，折叠或展开对应的菜单
$("#fold>ul>li").click(function () {
  $(this).siblings("li").find(".wrap").css({ display: "none" });
  $(this).find(".wrap").css({ display: "block" });
});
```

![折叠菜单](https://s3.ax1x.com/2021/03/14/603v5D.png)

# DOM 节点操作案例

## 左移右移

样式

```html
<style>
  select {
    width: 100px;
    height: 150px;
  }
  input[type="button"] {
    width: 50px;
  }
  #opt {
    margin: 90px 10px 0;
  }
  .box {
    width: 80%;
    margin: 0 auto;
    background-color: #999999;
  }
  .box div {
    float: left;
  }
</style>
```

DOM

```html
<div class="box">
  <div id="left">
    <p>可选项</p>
    <select multiple="multiple">
      <option>添加</option>
      <option>移动</option>
      <option>修改</option>
      <option>查询</option>
      <option>打印</option>
      <option>删除</option>
    </select>
  </div>
  <div id="opt">
    <input id="toRight" type="button" value=">" /><br />
    <input id="toLeft" type="button" value="<" /><br />
    <input id="toAllRight" type="button" value=">>" /><br />
    <input id="toAllLeft" type="button" value="<<" /><br />
  </div>
  <div id="right">
    <p>已选项</p>
    <select multiple="multiple"></select>
  </div>
</div>
```

JavaScript

```js
// 获取按钮添加单击事件，获取第一个下拉框中被选中的option添加到第二个下拉框
$("#toRight").click(function () {
  // 右移
  $("#right>select").append($("#left>select>option:selected"));
});
$("#toLeft").click(function () {
  // 左移
  $("#left>select").append($("#right>select>option:selected"));
});
$("#toAllRight").click(function () {
  // 全部右移
  $("#right>select").append($("#left>select>option"));
});
$("#toAllLeft").click(function () {
  // 全部左移
  $("#left>select").append($("#right>select>option"));
});
```

![左移右移](https://s3.ax1x.com/2021/03/14/60UGIx.png)

# 事件操作案例

## 手风琴效果

样式

```html
<style>
  ul {
    list-style: none;
    margin: 0;
    padding: 0;
  }
  div {
    width: 1200px;
    height: 400px;
    margin: 50px auto;
    overflow: hidden;
  }
  div li {
    width: 240px;
    height: 400px;
    float: left;
  }
</style>
```

DOM

```html
<div id="box">
  <ul>
    <li><img src="images/1.jpg" /></li>
    <li><img src="images/2.jpg" /></li>
    <li><img src="images/3.jpg" /></li>
    <li><img src="images/4.jpg" /></li>
    <li><img src="images/5.jpg" /></li>
  </ul>
</div>
```

JavaScript

```js
$("#box>ul>li").on({
  mouseover: function () {
    // 当前的li的所有的兄弟元素，并修改其宽度
    $(this).siblings("li").css("width", "60.5px");
    // 将当前li的宽度设置为图片的宽度
    $(this).css("width", "958px");
  },
  mouseout: function () {
    // 显示所有li的部分图片内容
    $("#box>ul>li").css("width", "240px");
  },
});
```

# 动画特效案例

## 轮播图

样式

```css
/* 热点图样式 */
.banner {
  position: relative;
  overflow: hidden;
  margin: 100px auto;
  width: 958px;
  height: 400px;
}
.banner ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
.hot {
  position: absolute;
  top: 0;
  left: 0;
}
.hot li {
  float: left;
}

/* 小圆点样式 */
.dot {
  position: absolute;
  bottom: 10px;
  width: 100%;
  text-align: center;
  font-size: 0;
}
.dot li {
  display: block;
  display: inline-block;
  margin: 0 5px;
  width: 15px;
  height: 15px;
  border-radius: 100%;
  background: rgba(145, 144, 144, 0.5);
  cursor: pointer;
}
.dot .on {
  background-color: #fff;
}
/* 左右翻页箭头样式 */
.arrow {
  display: none;
}
.arrow span {
  display: block;
  width: 50px;
  height: 100px;
  background: rgba(0, 0, 0, 0.6);
  color: #fff;
  text-align: center;
  font-size: 40px;
  line-height: 100px;
  cursor: pointer;
}
.arrow .prev {
  position: absolute;
  top: 50%;
  left: 0;
  margin-top: -50px;
}
.arrow .next {
  position: absolute;
  top: 50%;
  right: 0;
  margin-top: -50px;
}
```

DOM

```html
<div class="banner">
  <ul class="hot">
    <!--轮播图片-->
    <li>
      <a href="#"><img src="images/1.jpg" /></a>
    </li>
    <li>
      <a href="#"><img src="images/2.jpg" /></a>
    </li>
    <li>
      <a href="#"><img src="images/3.jpg" /></a>
    </li>
    <li>
      <a href="#"><img src="images/4.jpg" /></a>
    </li>
    <li>
      <a href="#"><img src="images/5.jpg" /></a>
    </li>
  </ul>
  <!--小圆点-->
  <ul class="dot">
    <li class="on"></li>
    <li></li>
    <li></li>
    <li></li>
    <li></li>
  </ul>
  <!-- 左右翻页箭头-->
  <div class="arrow">
    <span class="prev">&lt;</span><span class="next">&gt;</span>
  </div>
</div>
```

JavaScript

```js
$(function () {
  var i = 0; // 当前显示的图片索引
  var timer = null; // 定时器
  var delay = 1000; // 图片自动切换的间隔时间
  var width = 958; // 每张图片的宽度
  var speed = 400; // 动画时间
  // 复制列表中的第一个图片，追加到列表最后，设置ul的宽度为图片张数 * 图片宽度
  var firstimg = $(".hot li").first().clone();
  $(".hot")
    .append(firstimg)
    .width($(".hot li").length * width);
  // 1. 设置周期计时器，实现图片自动切换
  timer = setInterval(imgChange, delay);
  // 2. 鼠标移入，暂停自动播放，移出，开始自动播放
  $(".banner").hover(
    function () {
      clearInterval(timer);
    },
    function () {
      timer = setInterval(imgChange, delay);
    }
  );
  // 3. 鼠标划入圆点
  $(".dot li").mouseover(function () {
    i = $(this).index();
    $(".hot")
      .stop()
      .animate({ left: -i * width }, 200);
    dotChange();
  });
  // 4. 设置左右切换的箭头显示和隐藏
  $(".banner").hover(
    function () {
      $(".arrow").show();
    },
    function () {
      $(".arrow").hide();
    }
  );
  // 5. 向右箭头
  $(".next").click(function () {
    imgChange();
  });
  // 6. 向左箭头
  $(".prev").click(function () {
    --i;
    if (i == -1) {
      i = $(".hot li").length - 2;
      $(".hot").css({ left: -($(".hot li").length - 1) * width });
    }
    $(".hot")
      .stop()
      .animate({ left: -i * width }, speed);
    dotChange();
  });
  // 自动切换图片
  function imgChange() {
    ++i;
    isCrack();
    dotChange();
  }
  // 无缝轮播
  function isCrack() {
    if (i == $(".hot li").length) {
      i = 1;
      $(".hot").css({ left: 0 });
    }
    $(".hot")
      .stop()
      .animate({ left: -i * width }, speed);
  }
  // 自动切换对应的圆点
  function dotChange() {
    if (i == $(".hot li").length - 1) {
      $(".dot li").eq(0).addClass("on").siblings().removeClass("on");
    } else {
      $(".dot li").eq(i).addClass("on").siblings().removeClass("on");
    }
  }
});
```

<!-- <h1 style='font-size: 100px; color: darkcyan;'>jQuery 🎨</h1> -->

<!-- <span style='font-size: 50px'><span style="color: gold;">write less</span><span style='color: orange'> do more</span></span> -->
