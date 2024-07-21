---
title: Vue样式篇
toc: true
date: 2023-09-19 07:23:32
tags: vue
categories: 前端学习
thumbnail: https://z1.ax1x.com/2023/09/24/pPTomH1.png
cover: https://z1.ax1x.com/2023/09/24/pPTomH1.png
---

📦 Vue3

<!--more-->

# `style`

- 全称`v-bind:style`，简写`:style`

😃 如果样式需要浏览器前缀，Vue 会自动加上

也可以自己传递

```html
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

- 如果浏览器不需要前缀，结果就只有`display: flex`

**传递对象**

- 可用 _camelCase_ 或者 _kebab-cased_

使用`:style="{}"`或者是`:style="[]"`绑定的样式

> 记得值也需要用 `""`包住，React 中就不用。如果需要使用模板字符串，双引号换成**`**

常见使用：

```js
const activeColor = ref("red");
const fontSize = ref(30);

const styleObject = reactive({
  color: "red",
  fontSize: "13px",
});

const isBig = ref(false);
const bigActive = computed(() => {
  isBig ? { fontSize: "18px" } : { fontSize: "14px" };
});
```

```html
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>

<div :style="{ 'font-size': fontSize + 'px' }"></div>

<div :style="styleObject"></div>

<div :style="isBig"></div>
```

**传递数组**

在 JS 中写好的样式对象传递到行内的数组中

```html
<div :style="[baseStyles, overridingStyles]"></div>
```

在组件上使用`style`，组件内的根元素会继承组件上的样式。👉[class 透传](#组件上的-class)

# `class`

- 全称`v-bind:class`，简写`:class`

> 记得值也需要用 `""`包住

`class`和`:class`可同时存在，渲染后合并

```html
<div class="main-pane" :class="{ active: isActive }"></div>
```

- 样式类`active`是否存在，取决于`isActive`是否为 true
- 和`style`一样，支持绑定*对象*或者*计算属性*，也支持*数组*

绑定数组的时候，还可以在数组里面嵌套对象

```html
<div :class="[{ active: isActive }, errorClass]"></div>
```

## 组件上的 class

对于**单根组件**，组件上的`class`会和组件内根元素的合并

```html
<!-- 子组件模板 -->
<p class="foo bar">Hi!</p>
```

```html
<!-- 在使用组件时 -->
<MyComponent class="baz boo" />
```

渲染结果

```html
<p class="foo bar baz boo">Hi!</p>
```

对于**多根组件**，在组件内部需要使用`$attr.class`来指定哪个**根元素**来接收

```html
<!-- MyComponent 模板使用 $attrs 时 -->
<p :class="$attrs.class">Hi!</p>
<span>This is a child component</span>
```

```html
<MyComponent class="baz" />
```

要在 JS 中获取到，可以通过`useAttrs()`

在`setup`中

```vue
<script setup>
import { useAttrs } from "vue";

const attrs = useAttrs();
</script>
```

不用`setup`的话

```js
export default {
  setup(props, ctx) {
    // 透传 attribute 被暴露为 ctx.attrs
    console.log(ctx.attrs);
  },
};
```

> 不能用监听器来监听 attrs 的变化

# `<style>`标签

## css 预处理器

`<style lang='xx'></style>`：`xx`一般会有 _less_, _sass_(使用的时候传*scss*)。

```html
<style lang="scss">
  .example {
    color: red;
  }
</style>
```

## 样式作用域

`<style scoped></style>`：使得里面的样式仅在当前组件可用

- 在同一组件中，可和代表**全局**的`<style></style>`同时存在
- 谨慎使用后代选择器

```vue
<style scoped>
.example {
  color: red;
}
</style>

<template>
  <div class="example">hi</div>
</template>
```

转换为：

```html
<style>
  .example[data-v-f3f3eg9] {
    color: red;
  }
</style>

<template>
  <div class="example" data-v-f3f3eg9>hi</div>
</template>
```

- data-v-f3f3eg9 后面的哈希值根据文件的路径生成

> 使用了作用域之后，父组件样式不会影响子组件。不过为了布局考虑，子组件的根节点会受到影响

**深度选择器**：`:deep()`

在*scoped*作用域里面，要选择子组件里面的内容，使用伪类`:deep()`

```vue
<style scoped>
.a :deep(.b) {
  /* ... */
}

/* 将会编译为 */
.a[data-v-f3f3eg9] .b {
  /* ... */
}
</style>
```

> 括号里面的内容不需要用引号包起来

**插槽选择器**：`:slotted`

默认情况下，`scoped`不会影响到`<slot/>`渲染出来的内容，因为它们被认为是父组件所持有并传递进来的。

要指定插槽的额内容

```vue
<style scoped>
:slotted(div) {
  color: red;
}
</style>
```

**全局选择器**：`:global`

就是让当前指定的选择器配置的样式全局范围内生效

```vue
<style scoped>
:global(.red) {
  color: red;
}
</style>
```

## CSS Modules

`<style module="xxx">`被编译成 CSS Module

- 在 CSS Module 里的内容会变成对象
  - 直接`module`，使用默认名称`$style`
  - 自定义名称`module="classes"`
- 可在`<template>`和`<script setup>`中使用

在*模板*中使用

```vue
<template>
  <p :class="$style.red">This should be red</p>
</template>

<style module>
.red {
  color: red;
}
</style>
```

- 绑定的 *class*会进行哈希化

在 _setup_ 中使用

```js
import { useCssModule } from "vue";

// 在 setup() 作用域中...
// 默认情况下, 返回 <style module> 的 class
useCssModule();

// 具名情况下, 返回 <style module="classes"> 的 class
useCssModule("classes");
```

## CSS 中的 `v-bind`

在`<style>`中用来绑定`<script>`中的值

```vue
<script setup>
const theme = {
  color: "red",
};
</script>

<template>
  <p>hello</p>
</template>

<style scoped>
p {
  // 是表达式，就需要用引号包起来
  color: v-bind("theme.color");
}
</style>
```

# 动画

Vue 提供组件 `<Transition>` 和 `<TransitionGroup>` 来处理元素的

- 进入
- 离开
- 列表顺序变化

的过渡效果

- `<Transition>`，组件或元素**进入**或**离开**DOM 时应用动画
- `<TransitionGroup>`，在一个`v-for`列表的元素或组件被**插入**，**移动**，**删除**时应用动画

## `<Transition>`组件

> 仅支持单个元素或者一个单根组件

动画通过默认插槽传递给它包裹的元素或者组件上。触发条件：

- `v-if`
- `v-show`
- `<component>`切换的时候
- 改变特殊的`key`属性

整个动画过程：

1. Vue 会自动检测目标元素是否应用了 CSS 过渡或动画。则一些 `CSS过渡class`（`v-enter`那些）会在适当的时机被添加和移除。
2. 如果有作为监听器的 `JavaScript钩子`，这些钩子函数会在适当时机被调用。
3. 上面两点都没，那么 DOM 的插入、删除操作将在浏览器的下一个动画帧后执行。

🚗 可用的属性和事件

属性

```ts
interface TransitionProps {
  /**
   * 用于自动生成过渡 CSS class 名。
   * 例如 `name: 'fade'` 将自动扩展为 `.fade-enter`、
   * `.fade-enter-active` 等。
   */
  name?: string;
  /**
   * 是否应用 CSS 过渡 class。
   * 默认：true
   */
  css?: boolean;
  /**
   * 指定要等待的过渡事件类型
   * 来确定过渡结束的时间。
   * 默认情况下会自动检测
   * 持续时间较长的类型。
   */
  type?: "transition" | "animation";
  /**
   * 显式指定过渡的持续时间。
   * 默认情况下是等待过渡效果的根元素的第一个 `transitionend`
   * 或`animationend`事件。
   */
  duration?: number | { enter: number; leave: number };
  /**
   * 控制离开/进入过渡的时序。
   * 默认情况下是同时的。
   */
  mode?: "in-out" | "out-in" | "default";
  /**
   * 是否对初始渲染使用过渡。
   * 默认：false
   */
  appear?: boolean;

  /**
   * 用于自定义过渡 class 的 prop。
   * 在模板中使用短横线命名，例如：enter-from-class="xxx"
   */
  enterFromClass?: string;
  enterActiveClass?: string;
  enterToClass?: string;
  appearFromClass?: string;
  appearActiveClass?: string;
  appearToClass?: string;
  leaveFromClass?: string;
  leaveActiveClass?: string;
  leaveToClass?: string;
}
```

事件

- @before-enter
- @before-leave
- @enter
- @leave
- @appear
- @after-enter
- @after-leave
- @after-appear
- @enter-cancelled
- @leave-cancelled (v-show only)
- @appear-cancelled

### 基于 CSS 的过渡效果

#### CSS 过渡 class

[图片来自 Vue - Transition](https://cn.vuejs.org/assets/transition-classes.f0f7b3c9.png)

1. `v-enter-from`：进入动画的起始状态。在元素插入之前添加，在元素插入完成后的下一帧移除。

2. `v-enter-active`：进入动画的生效状态。应用于整个进入动画阶段。在元素被插入之前添加，在过渡或动画完成之后移除。这个 class 可以被用来定义进入动画的持续时间、延迟与速度曲线类型。

3. `v-enter-to`：进入动画的结束状态。在元素插入完成后的下一帧被添加 (也就是 `v-enter-from` 被移除的同时)，在过渡或动画完成之后移除。

4. `v-leave-from`：离开动画的起始状态。在离开过渡效果被触发时立即添加，在一帧后被移除。

5. `v-leave-active`：离开动画的生效状态。应用于整个离开动画阶段。在离开过渡效果被触发时立即添加，在过渡或动画完成之后移除。这个 class 可以被用来定义离开动画的持续时间、延迟与速度曲线类型。

6. `v-leave-to`：离开动画的结束状态。在一个离开动画被触发后的下一帧被添加 (也就是 `v-leave-from` 被移除的同时)，在过渡或动画完成之后移除。

使用`name`属性声明一个过渡效果名来代替前缀`v`:

```vue
<Transition name="fade">
  ...
</Transition>
```

- `name`可动态绑定

搭配 css 自己的 `tansition` 属性；也可以使用 css 的`animation`，一般都在`*-enter-active`和`*-leave`中使用

> 同时使用`transition`和`animation`，需要在标签中使用`type`属性传入两者之一，以告诉 Vue 需要关注哪种动画类型

```css
/*
  进入和离开动画可以使用不同
  持续时间和速度曲线。
*/
.fade-enter-active {
  transition: all 0.3s ease-out;
  /** 或者 */
  animation: bounce-in 0.5s;
}

.fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
  /** 或者 */
  animation: bounce-in 0.5s reverse;
}

.fade-enter-from,
.fade-leave-to {
  transform: translateX(20px);
  opacity: 0;
}

@keyframes bounce-in {
  /** */
}
```

🎇 动画效果会直接应用在子元素上，此外还可以选择指定更深层次的子元素来触发动画效果

```vue
<Transition name="nested">
  <div v-if="show" class="outer">
    <div class="inner">
      Hello
    </div>
  </div>
</Transition>
```

```css
/* 应用于嵌套元素的规则 */
.nested-enter-active .inner,
.nested-leave-active .inner {
  transition: all 0.3s ease-in-out;
}

.nested-enter-from .inner,
.nested-leave-to .inner {
  transform: translateX(30px);
  opacity: 0;
}
/* ... 省略了其他必要的 CSS */
```

还可以单独拿出来，添加一个过渡延迟，以获得交错效果

```css
/* 延迟嵌套元素的进入以获得交错效果 */
.nested-enter-active .inner {
  transition-delay: 0.25s;
}
```

通过向标签传递`duration`来指定过渡的持续时间，以确保这个延迟会在这个动画周期中生效（如果在`transitionend`和`animationend`时间结束了，延迟都没到来）

```vue
<Transition :duration="550">...</Transition>
<!-- 也可以传递一个对象来分别指定进入和离开 -->
<Transition :duration="{ enter: 500, leave: 800 }">...</Transition>
```

#### 自定义过渡 class

向 `<Transition>` 传递以下的 props 来指定自定义的过渡 class：

- enter-from-class
- enter-active-class
- enter-to-class
- leave-from-class
- leave-active-class
- leave-to-class

在使用第三方 CSS 动画库的时候非常有用

```vue
<!-- 假设你已经在页面中引入了 Animate.css -->
<Transition
  name="custom-classes"
  enter-active-class="animate__animated animate__tada"
  leave-active-class="animate__animated animate__bounceOutRight"
>
  <p v-if="show">hello</p>
</Transition>
```

### JavaScript 钩子

也可与听过监听`<Transition>`组件上的事件来实现动画

```html
<Transition
  @before-enter="onBeforeEnter"
  @enter="onEnter"
  @after-enter="onAfterEnter"
  @enter-cancelled="onEnterCancelled"
  @before-leave="onBeforeLeave"
  @leave="onLeave"
  @after-leave="onAfterLeave"
  @leave-cancelled="onLeaveCancelled"
>
  <!-- ... -->
</Transition>
```

```js
// 在元素被插入到 DOM 之前被调用
// 用这个来设置元素的 "enter-from" 状态
function onBeforeEnter(el) {}

// 在元素被插入到 DOM 之后的下一帧被调用
// 用这个来开始进入动画
function onEnter(el, done) {
  // 调用回调函数 done 表示过渡结束
  // 如果与 CSS 结合使用，则这个回调是可选参数
  done();
}

// 当进入过渡完成时调用。
function onAfterEnter(el) {}

// 当进入过渡在完成之前被取消时调用
function onEnterCancelled(el) {}

// 在 leave 钩子之前调用
// 大多数时候，你应该只会用到 leave 钩子
function onBeforeLeave(el) {}

// 在离开过渡开始时调用
// 用这个来开始离开动画
function onLeave(el, done) {
  // 调用回调函数 done 表示过渡结束
  // 如果与 CSS 结合使用，则这个回调是可选参数
  done();
}

// 在离开过渡完成、
// 且元素已从 DOM 中移除时调用
function onAfterLeave(el) {}

// 仅在 v-show 过渡中可用
function onLeaveCancelled(el) {}
```

使用仅由 JavaScript 执行的动画时，可添加一个 `:css="false"`，告诉 Vue 可跳过对 CSS 过渡的自动检测。使用之后：

- `@enter`和`@leave`回调函数`done`是必须的，否则钩子会被同步调用，过渡将立即完成

### 可复用

😃 可以向标签里面传递`<slot>`来封装成可复用的过渡组件

```vue
<!-- MyTransition.vue -->
<script>
// JavaScript 钩子逻辑...
</script>

<template>
  <!-- 包装内置的 Transition 组件 -->
  <Transition name="my-transition" @enter="onEnter" @leave="onLeave">
    <slot></slot>
    <!-- 向内传递插槽内容 -->
  </Transition>
</template>

<style>
/*
  必要的 CSS...
  注意：避免在这里使用 <style scoped>
  因为那不会应用到插槽内容上
*/
</style>
```

```html
<MyTransition>
  <div v-if="show">Hello</div>
</MyTransition>
```

### 其他过渡效果

**出现时过渡**

初次渲染时应用一个过渡效果

```template
<Transition appear>
  ...
</Transition>
```

**元素间过渡**

使用 `v-if` / `v-else` / `v-else-if`在几个组件之间切换

```html
<Transition>
  <button v-if="docState === 'saved'">Edit</button>
  <button v-else-if="docState === 'edited'">Save</button>
  <button v-else-if="docState === 'editing'">Cancel</button>
</Transition>
```

**过渡模式**

想先执行离开动画，等它完成之后在执行元素的进入动画

```vue
<Transition mode="out-in">
  ...
</Transition>
```

- 也支持`in-out`，但是不常用

**组件之间过渡**

```vue
<Transition name="fade" mode="out-in">
  <component :is="activeComponent"></component>
</Transition>
```

### 性能考虑

多数使用`transform`和`opacity`

1. 因为不会影响到 DOM 结构
2. 多数现代浏览器的`transform`动画会使用 GPU 进行硬件加速

相比之下`height`、`margin`这些会触发 CSS 布局变动，计算消耗就高了。[🌏 查看更多](https://csstriggers.com/)

## `<TransitionGroup>`组件

和`<Transition>`基本相同的属性、CSS 过渡 class、JavaScript 钩子

区别：

- 默认情况下不会生成容器元素；通过`tag`指定一个元素作为容器元素来渲染（外层自动生成的包裹层）
- _过渡模式_（`mode`）不可用
- 列表中每个元素需要有唯一`key`
- CSS 过渡 class 会被应用在列表上而不是容器元素上

> 在*DOM 模板*（html 文件或者 JS 片段）中使用时，需要使用`<transition-group>`

```vue
<TransitionGroup name="list" tag="ul">
  <li v-for="item in items" :key="item">
    {{ item }}
  </li>
</TransitionGroup>
```

```css
.list-move, /* 对移动中的元素应用的过渡 */
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

/* 确保将离开的元素从布局流中删除
  以便能够正确地计算移动的动画。 */
.list-leave-active {
  position: absolute;
}
```

事件和`<Transition>`一样

属性对比`<Transition>`少了`mode`，其他一样，并且多了两个：

```ts
interface TransitionGroupProps extends Omit<TransitionProps, "mode"> {
  /**
   * 如果未定义，则渲染为片段 (fragment)。
   */
  tag?: string;
  /**
   * 用于自定义过渡期间被应用的 CSS class。
   * 在模板中使用 kebab-case，例如 move-class="xxx"
   */
  moveClass?: string;
}
```

### 渐进延迟列表动画

- 通过读取元素的 data attribute

把每一个元素的索引渲染为该元素上的一个 data attribute

```vue
<TransitionGroup
  tag="ul"
  :css="false"
  @before-enter="onBeforeEnter"
  @enter="onEnter"
  @leave="onLeave"
>
  <li
    v-for="(item, index) in computedList"
    :key="item.msg"
    :data-index="index"
  >
    {{ item.msg }}
  </li>
</TransitionGroup>
```

在 JavaScript 钩子中，基于当前元素的 data attribute 对元素的进场动画添加一个延迟

```js
function onEnter(el, done) {
  gsap.to(el, {
    opacity: 1,
    height: "1.6em",
    delay: el.dataset.index * 0.15,
    onComplete: done,
  });
}
```
