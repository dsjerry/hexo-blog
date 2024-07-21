---
title: Vue Props篇
toc: true
date: 2023-10-17 22:09:45
tags: vue
categories: 前端学习
thumbnail: https://z1.ax1x.com/2023/10/19/piiqnSI.png
cover: https://z1.ax1x.com/2023/10/19/piiqnSI.png
---

🌤️ Vue3

<!--more-->

相比于组件内的状态 `ref` 和 `reactive`，`props` 用来在组件之间传值（父组件 -> 子组件）

# 声明

❗❗ `defineProps()`宏中的参数**不可访问**`<script setup>`中定义的其他变量，因为在编译时整个表达式都会被移到外部的函数中。

## 普通

🎇 在单文件组件的`<script setup>`中

```vue
<script setup>
// 单纯的声明
const props = defineProps(["foo"]);
console.log(props.foo);

// 声明时指定对应类型的 构造函数
const props2 = defineProps({
  title: { type: String, required: true },
  likes: Number,
});

props2.title; // string
props2.likes; // number | undefined
</script>
```

- 如果只在模板（`<template>`）中使用，可不用声明变量`props`，可直接在模板中使用`foo`

> 大写`String`和小写`string`不是同一个东西；大写是 JavaScript 中的**构造函数**，小写用在 TypeScript，是字符串类型

---

🎇 如果没有使用`<script setup>`

```js
export default {
  props: ["foo"],
  setup(props) {
    // setup() 接收 props 作为第一个参数
    console.log(props.foo);
  },
};
```

🎇 或者在`JSX`语法中

```tsx
import { defineComponent } from "vue";

export default defineComponent({
  props: { msg: { type: String, required: true } },
  setup(props: { msg: string }) {
    return () => <span class={"error-tips"}>{props.msg}</span>;
  },
});
```

## TS 中

🪄 上面使用构造函数声明 props 的叫做*运行时声明*，接下来用作泛型传递给`<>`的方式叫做*基于类型的声明*

- 两种方式不能同时使用

---

就像创建接口对象一样

```vue
<script setup lang="ts">
defineProps<{
  title: string;
  likes?: number;
}>();
</script>
```

可以直接提取成`interface`

```ts
interface Props {
  title: string;
  likes?: number;
}

definedProps<Props>();
```

> 🎇 多行书写时`;`可省略。由于限制，当前条件类型仅可指定单个属性，不能指定整个 props 对象

# 校验

普通模式中，检验不是必须的，不进行校验的话直接传递*字符串数组*就行：

```js
defineProps(["name", "age", "isShow"]);
```

## 普通

- `type`指定`props`类型
- `required`为`true`指定为必传 props，`false`代表默认可选
- `default`指定默认值

使用`type`指定类型，这些类型时原生 JavaScript 的构造函数：

| 🌸     | 🌺     | 🏵️       | 🪷      |
| ------ | ------ | -------- | ------ |
| String | Number | Boolean  | Array  |
| Object | Date   | Function | Symbol |

``

除此之外，可以通过自定义类或者构造函数来用作 props 的类型，Vue 会通过`instanceof`来检查：

```js
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
}

defineProps({
  author: Person,
});
```

- Vue 校验的时候会使用到`instanceof Person`

---

校验选项参考：

```js
defineProps({
  // 基础类型检查
  // （给出 `null` 和 `undefined` 值则会跳过任何类型检查）
  propA: Number,
  // 多种可能的类型
  propB: [String, Number],
  // 必传，且为 String 类型
  propC: {
    type: String,
    required: true,
  },
  // Number 类型的默认值
  propD: {
    type: Number,
    default: 100,
  },
  // 对象类型的默认值
  propE: {
    type: Object,
    // 对象或数组的默认值
    // 必须从一个工厂函数返回。
    // 该函数接收组件所接收到的原始 prop 作为参数。
    default(rawProps) {
      return { message: "hello" };
    },
  },
  // 自定义类型校验函数
  propF: {
    validator(value) {
      // The value must match one of these strings
      return ["success", "warning", "danger"].includes(value);
    },
  },
  // 函数类型的默认值
  propG: {
    type: Function,
    // 不像对象或数组的默认，这不是一个
    // 工厂函数。这会是一个用来作为默认值的函数
    default() {
      return "Default function";
    },
  },
});
```

## TS

使用到了 TypeScript 就说明是在校验了：

```ts
defineProps<{
  foo: string; // 字符串类型
  bar?: number; // 可选
}>();
```

默认值：需要使用`withDefaults`编译宏

```ts
export interface Props {
  msg?: string;
  labels?: string[];
}

const props = withDefaults(defineProps<Props>(), {
  msg: "hello",
  labels: () => ["one", "two"],
});
```

### 在非`<script setup>`的情况下

```ts
import { defineComponent } from "vue";

export default defineComponent({
  props: {
    message: String,
  },
  // setup的props参数会从上面推断，所以这里可以不用指定类型
  setup(props) {
    props.message; // <-- 类型：string
  },
});
```

### 复杂的 props 类型

🤔 刚接触的时候还挺容易混淆

简单的情况：

```vue
<script setup lang="ts">
interface Book {
  title: string;
  author: string;
  year: number;
}

const props = defineProps<Book>();
</script>
```

- props 将会有 title,author,year

复杂的情况，实则是**再嵌套**：

```vue
<script setup lang="ts">
interface Book {
  title: string;
  author: string;
  year: number;
}

const props = defineProps<{
  book: Book;
}>();
</script>
```

- 现在是 props 只有 book 一个，然后 book 下面才有这三个属性

## 即普通又 TS

想用一定的 TypeScript 类型检查，但是又不想传递范式（`<T>`），会这样使用。通常在在*Options API*中出现

需要导入**工具类型**`PropType`

```ts
import type { PropType } from "vue";

const props = defineProps({
  book: Object as PropType<Book>,
});
```

# 使用

🎇 在`<script>`中使用的时候推荐 camelCase 的命名方式；在模板中使用的时候推荐 kebab-case 的形式

传递 props（父组件向子组件传值）：

- 动态绑定值需要在属性前加上`v-bind`或者简写`:`
- 不加上的都认作是传递字符串

父组件：`<Page>`

```vue
<template>
  <div>
    <!-- 如果 likes 在这直接传递数字，并且想得到number类型 -->
    <!-- 可以动态绑定 v-bind="100" -->
    <MyTitle title="静夜思" :likes="likes" />
  </div>
</template>
<script setup lang="ts">
import { ref } from "vue";

const likes = ref(100);
</script>
```

子组件：`<MyTitle>`

```vue
<template>
  <h1>{{ title }}</h1>
  <p>{{ likes }}</p>
</template>
<script setup lang="ts">
defineProps<{
  title: string;
  likes?: number;
}>;
</script>
```

# 单向数据流

所有的 props 都遵循**单向绑定**原则

- 因父组件变化而变化，更新随之传向子组件，所以子组件中的 props 是最新的
- 子组件中不要修改 props。要想获得衍生值
  - 可以使用计算属性
  - 可以创建新的变量接收

当 props 是一个对象或者数组时，可以改变内部的值，但是**不推荐**

# Boolean 类型转换

方便在模板中使用时更加简洁，避免类似`:disabled='true'`的出现，使用简单的`disabled`表示即可

在`<MyComponent>`中：

```js
definedProps({
  disabled: Boolean,
});
```

使用时：

```html
<!-- 等同于传入 :disabled="true" -->
<MyComponent disabled />

<!-- 等同于传入 :disabled="false" -->
<MyComponent />
```

当 props 有多种类型的时候，这种转换依然适用。不过与`String`搭配时有例外。在排列时`Boolean`需要排在`String`的前面

```js
// disabled 将被转换为 true
defineProps({
  disabled: [Boolean, Number],
});

// disabled 将被转换为 true
defineProps({
  disabled: [Boolean, String],
});

// disabled 将被转换为 true
defineProps({
  disabled: [Number, Boolean],
});

// disabled 将被解析为空字符串 (disabled="")
defineProps({
  disabled: [String, Boolean],
});
```

# 泛型

使用`generic`属性，属性值和和在 TypeScript 中的`<T>`一样

```vue
<script setup lang="ts" generic="T">
defineProps<{
  items: T[];
  selected: T;
}>();
</script>
```

同时也可以使用`extends`进行泛型约束

```vue
<script setup lang="ts" generic="T extends string | number, U extends Item">
import type { Item } from "./types";
defineProps<{
  id: T;
  list: U[];
}>();
</script>
```

# provide 和 inject

不局限于相邻的父子组件传值，可以更深层次传递（React 中的*context*）

## `provide()`提供数据

```vue
<script setup>
import { ref, provide } from "vue";

const msg = ref("hello");
const setMsg = () => {
  msg.value = "hi";
};

provide(/* 注入名 */ "message", /* 值 */ { msg, setMsg });
</script>
```

- 一个组件可以多次调用，传递不同的注入名
- 如果传入的是一个普通变量，还可以使用`readonly()`将它包住

应用层 provide，全局提供数据。（在编写插件的时候会用到）

```js
import { createApp } from "vue";

const app = createApp({});

app.provide(/* 注入名 */ "message", /* 值 */ "hello!");
```

## `inject()`获取数据

```vue
<script setup>
import { inject } from "vue";

const { msg, setMsg } = inject("message", "这是默认值");
</script>
```

如果上游组件并没有提供`message`这样一个数据，给`inject`传递的第二个参数将会是作为当前返回的默认值

默写情况下为避免产生副作用，默认值可以是一个工厂函数，这时候第三个参数需要设置为`true`

```js
const value = inject("key", () => new ExpensiveClass(), true);
```

> 目前 TypeScript 标注好像作用不大

# 状态管理

1. 多个组件共享状态；

- 可通过 `props` 来解决，但是组件嵌套深了后容易混乱
- 可通过`provide`和`inject`

2. 不同的视图交互可修改同一状态

- 通过模板获取对应组件实例，然后触发组件内对应的事件

状态管理中最直接的方法，就是将对应的共享状态抽离出来

## 响应式 API 做状态管理

将共享的状态单独抽离出来

```js
// store.js
import { reactive } from "vue";

export const store = reactive({
  count: 0,
  increment() {
    this.count++;
  },
});
```

```vue
<!-- ComponentA.vue -->
<script setup>
import { store } from "./store.js";
</script>

<template>From A: {{ store.count }}</template>
```

```vue
<!-- ComponentB.vue -->
<script setup>
import { store } from "./store.js";
</script>

<template>From B: {{ store.count }}</template>
```

每当`store`被改动时，这两个组件都会自动更新自己的视图

不局限于`reactive`其他的响应式 API 同样适用

```js
import { ref } from "vue";

// 全局状态，创建在模块作用域下
const globalCount = ref(1);

export function useCount() {
  // 局部状态，每个组件都会创建
  const localCount = ref(1);

  return {
    globalCount,
    localCount,
  };
}
```

## 其他

1. 使用 `Pinia`
2. SSR ???
