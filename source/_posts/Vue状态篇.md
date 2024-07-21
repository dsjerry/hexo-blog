---
title: Vue响应式篇
toc: true
date: 2023-10-01 18:15:16
tags: vue
categories: 前端学习
thumbnail: https://z1.ax1x.com/2023/10/02/pPL4aSf.png
cover: https://z1.ax1x.com/2023/10/02/pPL4aSf.png
---

🌤️ Vue3

<!--more-->

# 声明变量

`ref()`和`reactive()`都是用来声明响应式的变量（代码改变页面也会跟着改变），两者不同在于：

- 前者用于基本原始值：string、number、boolean、null、undefined、symbol（下面的也能用）
- 后者用于：对象、数组、set、map

在`<script setup>`使用时，通过`ref()`声明的变量取值的时候需要使用`.value`

- 如果在`reactive()`里面传进一个`ref()`，不需要使用，会自动解包
- 但是`reactive()`里面是数组或者 map 的时候，又需要手动`.value`

```ts
import { ref, reactive } from "vue";

const age = ref(18);
const xm = reactive({ name: "xm", gender: "male", age });

console.log(age.value); // 18
console.log(xm.name); // 'xm'
console.log(xm.age); // 18

const books = reactive([ref("Vue 3 Guide")]);
const map = reactive(new Map([["count", ref(0)]]));

console.log(books[0].value); // 'Vue 3 Guide'
console.log(map.get("count").value); // 0
```

`reactive()`声明的对象重新赋值，会丢失响应性；在解构的时候，也会丢失响应性

```js
let state = reactive({ count: 0 });

// 上面的 ({ count: 0 }) 引用将不再被追踪
state = reactive({ count: 1 });

let { count } = state;
// 不会影响原始的 state
count++;
```

## 模板中使用

自动解包条件：

- 顶级的`ref()`会自动解包
- 不是顶级，但是只是取值（无其他 js 操作），也可自动解包

```js
const count = ref(0);
const object = { id: ref(1) };
```

```html
<!-- 正常运作 -->
<div>{{ count++ }}</div>
<!-- 失败，得到 [object Object]1 -->
<div>{{ object.id + 1 }}</div>
<!-- 正常运作，相当于在后面加上.value-->
<div>{{ object.id }}</div>
```

## 类型标注

一般指定`<script setup lang='ts'>`为 _TypeScript_ 时有自动的类型推断，不需要手动指定

🤔 指定`ref`的类型

```ts
import { ref } from "vue";
import type { Ref } from "vue";

const year: Ref<string | number> = ref("2020");
// 或者是
const year = ref<string | number>("2020");

year.value = 2020;
```

🤔 指定`reactive`的类型

```ts
import { reactive } from "vue";

interface Book {
  title: string;
  year?: number;
}

const book: Book = reactive({ title: "Vue 3 指引" });
```

# 说明

使用带`.value`的 ref 而不是普通的变量

- 渲染的时候**追踪**渲染过程中**用到的**每一个 ref
- ref 变化时，触发组件内的重新渲染函数
- 普通变量无法达到这个目的

类似于这样

```js
// 伪代码，不是真正的实现
const myRef = {
  _value: 0,
  get value() {
    track();
    return this._value;
  },
  set value(newValue) {
    this._value = newValue;
    trigger();
  },
};
```

> 不是每一次的 ref 变动都触发页面重新渲染。Vue 会在“next tick”更新周期中缓冲所有状态的修改，不管进行了多少次状态修改，每个组件都只会被更新一次。

要等待 DOM 更新完成再执行，需要使用`nextTick()`

```html
<script setup>
  import { ref, nextTick } from "vue";

  const count = ref(0);

  async function increment() {
    count.value++;
    // DOM 还未更新
    console.log(document.getElementById("counter").textContent); // 0
    // 可以 await，也可以传一个回调函数进去
    await nextTick();
    // DOM 此时已经更新
    console.log(document.getElementById("counter").textContent); // 1
  }
</script>

<template>
  <button id="counter" @click="increment">{{ count }}</button>
</template>
```

# 计算属性

`computed(()=>{})`，必须要有返回值；不能接收参数

- 和`ref()`一样，在 JS 中使用时需要`.value`

通过现有的变量来进一步计算获得新的值（临时值）；或者将条件判断写到计算属性中，简化模板中`{{}}`里面的内容

- 因为是在现有基础上得来的（派升值），所以直接修改它没什么一样，所以一般认为这是只读的

```html
<script setup>
  import { reactive, computed } from "vue";

  const author = reactive({
    name: "John Doe",
    books: [
      "Vue 2 - Advanced Guide",
      "Vue 3 - Basic Guide",
      "Vue 4 - The Mystery",
    ],
  });

  // 一个计算属性 ref
  const publishedBooksMessage = computed(() => {
    return author.books.length > 0 ? "Yes" : "No";
  });
</script>

<template>
  <p>Has published books:</p>
  <span>{{ publishedBooksMessage }}</span>
</template>
```

> 会自动跟踪响应式依赖。它会检测到 publishedBooksMessage 依赖于 author.books，所以当 author.books 改变时，任何依赖于 publishedBooksMessage 的绑定都会同时更新。

**TS 类型标注**

一般情况下会自动推断，手动指定：

```ts
const double = computed<number>(() => {
  // 若返回值不是 number 类型则会报错
});
```

**与方法的区别**

计算属性会缓存，渲染时如果所依赖的变量没有改变，计算属性不会执行（不执行 getter 函数）。相比之下，方法在每次渲染的时候都会执行。

**计算属性是基于响应式依赖被缓存**，下面`Date.now()`不是响应式的，所以不会被缓存

```js
const now = computed(() => Date.now());
```

## 可写计算属性

```js
<script setup>
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  // getter
  get() {
    return firstName.value + ' ' + lastName.value
  },
  // setter
  set(newValue) {
    // 注意：我们这里使用的是解构赋值语法
    [firstName.value, lastName.value] = newValue.split(' ')
  }
})
</script>
```

> getter 不应该有副作用（变更现有值，触发 DOM 重新渲染）

# 监听器

`watch(ref, (newVal, oldVal)=>{})`

- 返回一个函数，再次调用可停止当前监听

相比于 _计算属性_，监听器做的是监听到值变化的时候做一些副作用（DOM 页面更新）。[watch api](https://cn.vuejs.org/api/reactivity-core.html#watch)

```js
import { ref, watch } from "vue";

const x = ref(0);
const y = ref(0);

// 单个 ref
watch(x, (newX) => {
  console.log(`x is ${newX}`);
});

// getter 函数
watch(
  () => x.value + y.value,
  (sum) => {
    console.log(`sum of x + y is: ${sum}`);
  }
);

// 多个来源组成的数组
watch([x, () => y.value], ([newX, newY]) => {
  console.log(`x is ${newX} and y is ${newY}`);
});
```

监听`reactive`对象的时候，不能直接监听它的属性，而是需要一个 getter 函数

```js
const obj = reactive({ count: 0 });

// 错误，因为 watch() 得到的参数是一个 number
watch(obj.count, (count) => {
  console.log(`count is: ${count}`);
});

// 提供一个 getter 函数
watch(
  () => obj.count,
  (count) => {
    console.log(`count is: ${count}`);
  }
);
```

## 深度监听

监听响应式对象的时候，会**隐式地**进行深度监听

```js
const obj = reactive({ count: 0 });

watch(obj, (newValue, oldValue) => {
  // 在嵌套的属性变更时触发
  // 注意：`newValue` 此处和 `oldValue` 是相等的
  // 因为它们是同一个对象！
});

obj.count++;
```

如果只需要监听对象的改变，需要监听一个 getter 函数

```js
watch(
  () => state.someObject,
  () => {
    // 仅当 state.someObject 被替换时触发
  }
);
```

在使用 getter 函数的基础上，也可以进行强制深度监听

```js
watch(
  () => state.someObject,
  (newValue, oldValue) => {
    // 注意：`newValue` 此处和 `oldValue` 是相等的
    // *除非* state.someObject 被整个替换了
  },
  { deep: true }
);
```

## 回调触发时机

指定`flush`的值，有：`'pre' | 'post' | 'sync'`

**默认情况**（pre）中，是在 DOM 更新前触发的，所以在回调里面获得的 DOM 是旧的

**DOM 更新后触发**（post）

```js
import { watchEffect, watchPostEffect } from "vue";

watch(source, callback, {
  flush: "post",
});

watchEffect(callback, {
  flush: "post",
});

watchPostEffect(() => {
  /* 在 Vue 更新后执行. 对应的是 flush: 'post' */
});
```

- `watchSyncEffect()`为同步触发，对应的`watchEffect`为`flush: 'sync'`

**立即触发**

`watch`默认是懒执行的：仅当数据源变化时，才会执行回调。

如果需要在创建的时候立即执行一次：

```js
watch(
  source,
  (newValue, oldValue) => {
    // 立即执行，且当 `source` 改变时再次执行
  },
  { immediate: true }
);
```

## watchEffect

和`wacth`的区别是不用指定要监听的目标，它会自动跟踪（跟踪取值的那个变量）

- 不需要指定`immediate: true`也会立即执行

```js
const todoId = ref(1);
const data = ref(null);

watch(
  todoId,
  async () => {
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
    );
    data.value = await response.json();
  },
  { immediate: true }
);
```

使用`watchEffect`简化

```js
watchEffect(async () => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
  );
  data.value = await response.json();
});
```

- 对于需要监听多个依赖的时候非常有用

## 停止监听器

在`setup`中**同步语句**创建的监听器，会随着组件卸载而自动停止。如果是在**异步语句**中创建，需要手动停止

```html
<script setup>
  import { watchEffect } from "vue";

  // 它会自动停止
  watchEffect(() => {});

  // ...这个则不会！
  setTimeout(() => {
    watchEffect(() => {});
  }, 100);
</script>
```

调用监听器执行后的返回值停止监听器

```js
const unwatch = watchEffect(() => {});

// ...当该侦听器不再需要时
unwatch();
```

> 一般都是同步创建监听器的 😃

# 其他 API

`isRef()`

判断变量是不是*ref*

`shallowRef()`

ref 的浅层形式，只对第一个`.`保持响应式。改变深层次的`.`，页面不会更新；在处理大型数据结构的时候对性能有帮助

```js
const state = shallowRef({ count: 1 });

// 不会触发更改
state.value.count = 2;

// 会触发更改
state.value = { count: 2 };
```

`triggerRef()`

强制触发一个`shalloRef()`。就是强制地将传进去的浅层 ref 变为深层次也有响应式

```js
const shallow = shallowRef({
  greet: "Hello, world",
});

// 触发该副作用第一次应该会打印 "Hello, world"
watchEffect(() => {
  console.log(shallow.value.greet);
});

// 这次变更不应触发副作用，因为这个 ref 是浅层的
// 如果关联页面，页面不会渲染
shallow.value.greet = "Hello, universe";

// 打印 "Hello, universe"（watchEffect监听到了变动）
triggerRef(shallow);
```

`customRef()`

有 _Proxy_ 的意思，在设置值取值之间进行自定义操作

- 接收一个回调，有两个参数
  - `track()`函数和`trigger()`函数
- 返回一个对象
  - 包含`get()`方法和`set()`方法
- 一般 track 在 get 里面调用；trigger 在 set 里面调用

例如

```js
import { customRef } from "vue";

export function useDebouncedRef(value, delay = 200) {
  let timeout;
  return customRef((track, trigger) => {
    return {
      get() {
        track();
        return value;
      },
      set(newValue) {
        clearTimeout(timeout);
        timeout = setTimeout(() => {
          value = newValue;
          trigger();
        }, delay);
      },
    };
  });
}
```

```vue
<script setup>
import { useDebouncedRef } from "./debouncedRef";
const text = useDebouncedRef("hello");
</script>

<template>
  <input v-model="text" />
</template>
```

---

`shallowReactive()`

如果里面传的是`ref()`，不会自动解包

```js
const state = shallowReactive({
  foo: 1,
  nested: {
    bar: 2,
  },
});

// 更改状态自身的属性是响应式的
state.foo++;

// ...但下层嵌套对象不会被转为响应式
isReactive(state.nested); // false

// 不是响应式的
state.nested.bar++;
```

`readonly()`

接受一个对象 (不论是响应式还是普通的)，返回一个只读对象代理

`shallowReadonly()`

只是第一个`.`只读

`toRaw()`

返回由 `reactive()`、`readonly()`、`shallowReactive()` 或者 `shallowReadonly()` 创建的代理对应的原始对象

```js
const foo = {};
const reactiveFoo = reactive(foo);

console.log(toRaw(reactiveFoo) === foo); // true
```

`markRaw()`

将对象标记为不可代理（不能设置为响应式）。返回对象本身

```js
const foo = markRaw({});
console.log(isReactive(reactive(foo))); // false

// 也适用于嵌套在其他响应性对象
const bar = reactive({ foo });
console.log(isReactive(bar.foo)); // false
```

`effectScope()`

创建一个 effect 作用域，捕获创建的副作用（计算属性和监听器），然后同时处理

```js
const scope = effectScope();

scope.run(() => {
  const doubled = computed(() => counter.value * 2);

  watch(doubled, () => console.log(doubled.value));

  watchEffect(() => console.log("Count: ", doubled.value));
});

// 处理掉当前作用域内的所有 effect
scope.stop();
```

`getCurrentScope()`

如果有的话，返回当前活跃的 effect 作用域。

`onScopeDispose()`

接收一个回调，当 effect 作用域停止的时候会执行这个回调。`onUnmounted`的替代品
