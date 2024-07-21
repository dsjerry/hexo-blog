---
title: Pinia快速手册
toc: true
date: 2023-09-24 19:44:45
tags: [vue, 状态管理]
categories: 前端学习
thumbnail: https://z1.ax1x.com/2023/09/30/pPqd9LF.png
cover: https://z1.ax1x.com/2023/09/30/pPqd9LF.png
---

🌏 Vue 状态管理

<!-- more -->

# 启动

## 概念

`Store`：不与组件树绑定，承载着全局状态（数据）。在`Vue`中有`vuex`，`pinia`；在`React`中，有`redux`、`unstate-next`

🤔 并非所有应用都需要全局状态

🏞️ Store 的使用场景：在整个应用中可访问到的数据。例如：

- 导航栏的用户信息
- 复杂的多步骤表单

> 避免将本可以保存至组件中的数据保存到 Store，例如一个元素在页面的可见性

对于`pinia`，有**state**、**getter**、**action**三个概念

可理解为对应组件的

- `data` -> `state`
- `computed` -> `getter`
- `methods` -> `action`

在 setup 中的对应关系是

- `state` -> `ref()`
- `getter` -> `computed()`
- `action` -> `function()`

这三个概念可以认为是：值、通过值得到的、用来改变值的

## 安装

通过 `Vite` 创建一个 `Vue` 项目

```shell
pnpm create vite
```

安装`pinia`

```shell
pnpm add pinia
```

习惯在相对项目根目录创建`src/store/index.ts`文件，以此作为入口来配置整个项目的 _store_（state？？）

```ts
import { createPinia } from "pinia";

const pinia = createPinia();

export default pinia;
```

在`vue`的入口文件`main.ts`中导入并使用

```ts
import { createApp } from "vue";
import App from "./App.vue";
import pinia from "./store";

const app = createApp(App);

app.use(pinia);
app.mount("#app");
```

- 记得要在挂在`#app`之前使用

如果是在`Vue2`中

```ts
import { createPinia, PiniaVuePlugin } from "pinia";

Vue.use(PiniaVuePlugin);
new Vue({
  el: "#app",
  // 同一个`pinia'实例，可以在同一个页面的多个 Vue 应用中使用。
  pinia,
});
```

# 快速使用

## 在 setup 中使用

### 定义

- Composition API

在`setup`中使用非常方便，写法和普通的 JavaScript 大差不差

使用`defineStore`定义一个`Store`:（可在`src/store`下创建新的文件）

```ts
export const useUserStore = defineStore("user", () => {
  const username = ref("");
  const setUsername = (name: string) => {
    username.value = name;
  };

  return { username, setUsername };
});
```

🎇 Store 的名字推荐是`useXXXStore`

- 第一个参数是 Store 的唯一 ID
- 第二个参数是一个函数

### 使用

在 Vue 组件的`<script setup>`中

```js
import { useUserStore } from "xx/store/userStore";

// 直接解构的 username 不具有响应式
// 作为 action 的 setUsername 正常使用
const { username, setUsername } = useUserStore();

setUsername("Jerry");

console.log(username); // 还是原来的值
console.log(useUserStore().username); // Jerry
```

要保持响应性，可使用`storeToRefs`

```js
import { storeToRefs } from "pinia";

const store = useUserStore();
const { username } = storeToRefs(store);
// 需要变为响应性的是 state（值），action直接解构就行
const { setUsername } = store;
```

> 如果有报错说类似是：`setUsername is not a function`的，重启一下项目就好了。？不知道是 vite 的问题还是 pinia 的问题

## 不在 setup 中使用

- Option API

```ts
export const useCounterStore = defineStore("counter", {
  state: () => ({ count: 0 }),
  getters: {
    double: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++;
    },
  },
});
```

使用方法和在`setup`中一样，就像是`Vue`中*Option API*和*Composition API*的区别

# 细节使用

## state

```ts
import { defineStore } from "pinia";

const useStore = defineStore("storeId", {
  // 为了完整类型推理，推荐使用箭头函数
  state: (): State => {
    return {
      count: 0,
      items: [] as ItemInfo[],
    };
  },
});

interface ItemInfo {
  id: number;
  name: string;
}

interface State {
  count: string;
  items: array;
}
```

- 一般都会自动推断类型，无需手动写

### 修改

使用的时候可以直接进行读写来改变 store，也可以使用`$patch`方法

```js
const store = useStore();

store.count++;

store.$patch({
  count: store.count + 1,
  items: [{ id: 0, name: "huawei" }],
});

// 也可以以函数作为参数的形式使用
store.$patch((state) => {
  state.items.push({ name: "shoes", quantity: 1 });
  state.hasChanged = true;
});

// 可以重置state
store.$reset();
```

### 替换

**不能完全替换掉**store 的 state，可以 patch 它们

```js
// 这实际上并没有替换`$state`
store.$state = { count: 24 };
// 在它内部调用 `$patch()`：
store.$patch({ count: 24 });
```

可以通过变更`pinia`实例的`state`来设置整个应用的初始 state。常用于*SSR 中的激活过程*（为了安全起见，`pinia.state.value`被转义为其他形式）

```js
pinia.state.value = {};
```

### 订阅 State

`$subscribe`监听`state`的变化，相比于`watch`，好处是 _subscriptions_ （订阅的内容）在 _patch_ 后只触发一次

```js
cartStore.$subscribe((mutation, state) => {
  // import { MutationType } from 'pinia'
  mutation.type; // 'direct' | 'patch object' | 'patch function'
  // 和 cartStore.$id 一样
  mutation.storeId; // 'cart'
  // 只有 mutation.type === 'patch object'的情况下才可用
  mutation.payload; // 传递给 cartStore.$patch() 的补丁对象。

  // 每当状态发生变化时，将整个 state 持久化到本地存储。
  localStorage.setItem("cart", JSON.stringify(state));
});
```

_state subscription_ 会被绑定到添加它们的组件上，组件被卸载的时候，这些订阅会被删除。添加`{detachd: true}`作为第二个参数将订阅从当前组件中分离

```vue
<script setup>
const someStore = useSomeStore();
// 此订阅器即便在组件卸载之后仍会被保留
someStore.$subscribe(callback, { detached: true });
</script>
```

> 也可以在*pinia*实例上监听整个 state

```js
watch(
  pinia.state,
  (state) => {
    // 每当状态发生变化时，将整个 state 持久化到本地存储。
    localStorage.setItem("piniaState", JSON.stringify(state));
  },
  { deep: true }
);
```

### 在 OptionAPI 的用法

```js
// 示例文件路径：
// ./src/stores/counter.js

import { defineStore } from "pinia";

const useCounterStore = defineStore("counter", {
  state: () => ({
    count: 0,
  }),
});
```

使用`mapState()`辅助函数将 state 属性映射为**只读**的计算属性

```js
import { mapState } from 'pinia'
import { useCounterStore } from '../stores/counter'

export default {
  computed: {
    // 可以访问组件中的 this.count
    // 与从 store.count 中读取的数据相同
    ...mapState(useCounterStore, ['count'])
    // 与上述相同，但将其注册为 this.myOwnName
    ...mapState(useCounterStore, {
      myOwnName: 'count',
      // 你也可以写一个函数来获得对 store 的访问权
      double: store => store.count * 2,
      // 它可以访问 `this`，但它没有标注类型...
      magicValue(store) {
        return store.someGetter + this.count + this.double
      },
    }),
  },
}
```

使用`mapWritableState()`辅助函数将 state 映射为**可修改**

```js
import { mapWritableState } from 'pinia'
import { useCounterStore } from '../stores/counter'

export default {
  computed: {
    // 可以访问组件中的 this.count，并允许设置它。
    // this.count++
    // 与从 store.count 中读取的数据相同
    ...mapWritableState(useCounterStore, ['count'])
    // 与上述相同，但将其注册为 this.myOwnName
    ...mapWritableState(useCounterStore, {
      myOwnName: 'count',
    }),
  },
}
```

- 但是就不能传递函数了

## Getter

- 完全等同于 state 的计算属性。推荐使用箭头函数，接收一个 state 作为第一个参数
- 在`setup`中，就是通过 state 返回衍生值的那个方法（不传参）
- 不可异步

```js
export const useStore = defineStore("main", {
  state: () => ({
    count: 0,
  }),
  getters: {
    doubleCount: (state) => state.count * 2,
  },
});
```

### 访问其他 Getter

除了依赖 state，也可以访问到其他的 getter

在使用`TypeScript`的时候，如果是通过`this`访问其他 getter，需要明确当前 getter 的返回类型（ts 的问题）

```ts
export const useStore = defineStore("main", {
  state: () => ({
    count: 0,
  }),
  getters: {
    // 自动推断出返回类型是一个 number
    doubleCount(state) {
      return state.count * 2;
    },
    // 返回类型**必须**明确设置
    doublePlusOne(): number {
      // 整个 store 的 自动补全和类型标注 ✨
      return this.doubleCount + 1;
    },
  },
});
```

- 访问另外一个 store（另外的`defineStore`） 的 Getter，也是一样的操作

### 传参

计算属性不传参的，不过可以让 getter 返回一个函数，这个函数可以接收任意参数

```js
export const useStore = defineStore("main", {
  getters: {
    getUserById: (state) => {
      return (userId) => state.users.find((user) => user.id === userId);
    },
  },
});
```

- 这样的 getter 不会被缓存（计算属性缓存），但性能会好点

### 在`setup`中使用

作为 store 的一个属性，可直接访问

```vue
<script setup>
const store = useCounterStore();
store.count = 3;
store.doubleCount; // 6
</script>
```

### 在 OptionAPI 中的用法

这里面也分为两种风格：

1. 组合式 API，不是在`<script setup>`里面，是在`defineComponent`里面

```js
<script>
import { useCounterStore } from '../stores/counter'

export default defineComponent({
  setup() {
    const counterStore = useCounterStore()

    return { counterStore }
  },
  computed: {
    quadrupleCounter() {
      return this.counterStore.doubleCount * 2
    },
  },
})
</script>
```

2. OptionAPI 的形式（和 state 一样，使用辅助函数）

```js
import { mapState } from "pinia";
import { useCounterStore } from "../stores/counter";

export default {
  computed: {
    // 允许在组件中访问 this.doubleCount
    // 与从 store.doubleCount 中读取的相同
    ...mapState(useCounterStore, ["doubleCount"]),
    // 与上述相同，但将其注册为 this.myOwnName
    ...mapState(useCounterStore, {
      myOwnName: "doubleCount",
      // 你也可以写一个函数来获得对 store 的访问权
      double: (store) => store.doubleCount,
    }),
  },
};
```

## Action

- 相当于组件的`method`
- 在`setup`中，就是改变 state 的那个方法
- 可以是异步的

```js
import { mande } from "mande";

const api = mande("/api/users");

export const useUsers = defineStore("users", {
  state: () => ({
    userData: null,
    // ...
  }),

  actions: {
    async registerUser(login, password) {
      try {
        this.userData = await api.post({ login, password });
        showTooltip(`Welcome back ${this.userData.name}!`);
      } catch (error) {
        showTooltip(error);
        // 让表单组件显示错误
        return error;
      }
    },
  },
});
```

- 使用的时候正常调用就行

### 订阅 action

通过`store.$onAction()`来监听 action 和它的结果；传递给它的回调函数会在 action 本身之前执行

```js
const unsubscribe = someStore.$onAction(
  ({
    name, // 使用到的 action 名称
    store, // store 实例，类似 `someStore`
    args, // 传递给 action 的参数数组
    after, // 在 action 返回或解决后的钩子
    onError, // action 抛出或拒绝的钩子
  }) => {
    // 为这个特定的 action 调用提供一个共享变量
    const startTime = Date.now();
    // 这将在执行 "store "的 action 之前触发。
    console.log(`Start "${name}" with params [${args.join(", ")}].`);

    // 这将在 action 成功并完全运行后触发。
    // 它等待着任何返回的 promise（action的返回值）
    after((result) => {
      console.log(
        `Finished "${name}" after ${
          Date.now() - startTime
        }ms.\nResult: ${result}.`
      );
    });

    // 如果 action 抛出或返回一个拒绝的 promise，这将触发
    onError((error) => {
      console.warn(
        `Failed "${name}" after ${Date.now() - startTime}ms.\nError: ${error}.`
      );
    });
  }
);

// 手动删除监听器
unsubscribe();
```

- 订阅默认绑定在*添加它们的组件内*，组件卸载时，订阅也会自动取消

如果不想让订阅跟着组件取消，将`true`传递给第二个参数

```vue
<script setup>
const someStore = useSomeStore();
// 此订阅器即便在组件卸载之后仍会被保留
someStore.$onAction(callback, true);
</script>
```

### 在 OptionAPI 中的用法

1. 使用 `setup()`

```js
<script>
import { useCounterStore } from '../stores/counter'
export default defineComponent({
  setup() {
    const counterStore = useCounterStore()
    return { counterStore }
  },
  methods: {
    incrementAndPrint() {
      this.counterStore.increment()
      console.log('New Count:', this.counterStore.count)
    },
  },
})
</script>
```

2. 不使用 `setup()`，也是使用辅助函数

```js
import { mapActions } from 'pinia'
import { useCounterStore } from '../stores/counter'

export default {
  methods: {
    // 访问组件内的 this.increment()
    // 与从 store.increment() 调用相同
    ...mapActions(useCounterStore, ['increment'])
    // 与上述相同，但将其注册为this.myOwnName()
    ...mapActions(useCounterStore, { myOwnName: 'increment' }),
  },
}
```

# 插件

插件是**一个函数**，通过`pinia.use()`添加到 pinia 实例中，可以**选择性地返回**要添加到 store 的属性

```js
import { createPinia } from "pinia";

const pinia = createPinia();

pinia.use(() => {
  // 创建的每个 store 中都会添加一个名为 `secret` 的属性。
  return { secret: "the cake is a lie" };
});

// 在另一个文件中
const store = useStore();
store.secret; // 'the cake is a lie'
```

- 所以在创建全局（共享）的 store 变量时很有用
- 每个 store 都被`reactive`包装过，所以对于`ref`的值也无需使用`.value`

插件函数有一个**可选**的`context`参数

```js
export function myPiniaPlugin(context) {
  context.pinia; // 用 `createPinia()` 创建的 pinia。
  context.app; // 用 `createApp()` 创建的当前应用(仅 Vue 3)。
  context.store; // 该插件想扩展的 store
  context.options; // 定义传给 `defineStore()` 的 store 的可选对象。
  // ...
}
```

因为插件本身是一个函数，所以在`pinia.use`的时候也可以传递参数进去

🤔 第一个参数是一个*context*，要想接收到传递过来的参数，需要使用*函数柯里化*

- 在外面再包一层函数用来接收传递过来的参数
- 然后返回一个函数，作为参数传递给`pinia.use`

```js
export function myPlugin(options) {
  return (context) => {
    // 插件操作
  };
}
```

使用插件

```js
import { myPlugin } from "xx.js";

pinia.use(myPlugin({ msg: "hello pinia" }));
```

在 TypeScript 中使用时可以指定`options`的类型，然后再使用的时候会有类型检测

```ts
interface Options {
  msg: string;
}

export function myPlugin(options: Options) {
  return (context) => {
    // 插件操作
  };
}
```

在使用的时候传递不符合`Options`时会报错
