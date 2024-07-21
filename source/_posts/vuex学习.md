---
title: Vuex学习
date: 2020-07-18 16:16:45
tags: [vue]
categories: 前端学习
thumbnail: https://s3.ax1x.com/2021/01/23/s79KUK.jpg
cover: https://s3.ax1x.com/2021/01/23/s79KUK.jpg
toc: true
---

Vuex 是实现组件全局状态（数据）管理的一种机制，可以方便地实现组件之间数据的共享

传统组件之间共享数据的方式：

1. 父组件向子组件传值：v-bind 属性绑定
2. 子组件向父组件传值：v-on 事件绑定
3. 兄弟组件之间共享数据：EventBus
   - $on：接收数据的组件
   - $emit：发送数据的组件<!-- more -->

使用 Vuex 这个状态管理模式：

1. 在 Vuex 中集中管理共享的数据，易于开发和后期维护
2. 能够高效地实现组件之间的数据共享，提高开发效率
3. 存储在 Vuex 中的数据都是**响应式的**，能够实时保持数据与页面的同步

什么时候应该使用 Vuex ？

1. 一般情况下组件之间共享数据才有必要使用 Vuex，如果是组件的私有数据，保存在组件自己的 `data` 中就好了
2. 如果不是大型的单页面应用，使用 Vuex 可能是繁琐冗余的。简单的项目就不用使用了。

# 基本使用

安装 Vuex 依赖包

`npm install vuex --save`

导入

```javascript
import Vuex from "vuex";
Vue.use(Vuex);
```

创建 `store` 对象

```javascript
const store = new Vuex.Store({
  state: {
    count: 0,
  },
});
```

将 store 对象挂载到 Vue 实例中

```javascript
new Vue({
  el: "#app",
  render: (h) => h(app),
  router,
  store,
});
```

> 也可以在脚手架直接创建

# 核心概念

核心概念有：state、mutation、getter、action。要访问它们，各自都有两种方法，第一种是正常使用，第二种是在`要使用数据的组件`中导入 mapXXX 函数

## State

state 提供唯一的公共数据源，所有共享的数据放到 store 中的 state 中。`类似于 data`

- 在 Vuex 的入口文件中创建 Vuex 对象后，创建 Vuex 对象

组件中要访问 state 中数据的

**方法 ①**：

```javascript
this.$store.state.全局数据名称;
```

- 在插值表达式中使用上面代码时，`this` 可以省略

  ```html
  <h1>数值：{{ $store.state.count }}</h1>
  ```

**方法 ②**

从 Vuex 中按需导入`mapState` 函数

```javascript
import { mapState } from "vuex";
```

然后通过这个导入的函数，将当前组件需要的全局数据映射为当前组件的计算属性 **computed**

```javascript
computed: {
    ...mapState(['count'])
}
```

> `...` 对象展开运算符
>
> - `mapState` 返回的是一个对象，要将它与局部计算属性混合使用，就需要将多个对象合并成一个。所以这里使用**对象展开运算符**

> 对象展开运算符（不知道怎么翻译，就这样吧）
>
> - Reset Properties
>
>   ```javascript
>   let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
>   // x:1, y:2, z: {a: 3, b:4}
>   ```
>
> - Spread Properties
>
>   ```javascript
>   let n = { x, y, ...z };
>   n; // {x: 1, y: 2, a: 3, b: 4}
>   ```

**方法 ② 例子**

- 在需要的组件中引入 `mapState`

  ```javascript
  import { mapState } from "vuex";
  ```

- 在当前组件中的计算属性定义

  ```javascript
  ...mapState( ["count"] )
  ```

- 在 html 中使用（直接使用）

  ```html
  <h3>{ count }</h3>
  ```

## Mutation

更改 Vuex 的 store 中的状态（数据）的唯一方法是提交 mutation。`类似于事件`

### 定义

在 Vuex 的入口文件中

```javascript
const store = new Vuex.Store({
    state: {
        count: 0
    },
    mutations: {
        // 没有参数
        add(state) {
            state.count++
        }，
        // 携带参数
        addN(state, step){
    		state.count += step
		}
    }
})
```

### 触发

有两种方法：`this.$store.commit()` 和 `mapMutations`

**方法 ①**

在 _要使用到的组件_ 里面

- 里面先要有对应的事件处理函数，比如 addBtn 事件

  ```javascript
  methods: {
     addBtn() {
         this.$store.commit("add")
     },
     addNBtn() {
         this.$store.commit("addN", 3)
     }
  }
  ```

  > commit 里面对应的参数就是在 mutation 里面定义的内容

**方法 ②**

从需要的组件中按需导入 `mapMutations` 函数

```javascript
import { mapMutations } from "vuex";
```

通过展开运算符将 mutations 中的函数映射为当前组件中的 methods

```javascript
methods: {
    ...mapMutations( ['add', 'addN'] )
}
```

然后在 methods 中加入对应的时间处理函数，比如是页面的 addBtn 和 addNBtn

```javascript
methods: {
    ...mapMutations( ['add', 'addN'] ),
    addBtn() {
        this.add()
    },
    addNBtn() {
        this.addN(3)
    }
}
```

> 其实是可以直接调用映射来的方法的，因为它本来就是映射过来的方法，不用在另外的方法里面调用也行

## Action

Action 类似于 Mutation，但用于处理异步操作，是通过触发 Mutation 来修改数据

### 定义

```javascript
const store = new Vuex.Store({
  state: {
    count: 0,
  },
  mutations: {
    add(state) {
      state.count++;
    },
    addN(state, step) {
      state.count += step;
    },
  },
  actions: {
    addAsync(context) {
      setTimeOut(() => {
        context.commit("add");
      }, 1000);
    },
    addNAsync(context, step) {
      setTimeOut(() => {
        context.commit("addN", step);
      }, 1000);
    },
  },
});
```

- 这里的 commit 中的函数只能是在 mutation 中的函数，使用 `context` 来调用

### 触发

**方法 ①**

在需要使用的组件中

```javascript
methods: {
    addBtn() {
        this.$store.dispatch("addAsync")
    },
    addNBtn() {
        this.$store.dispatch("addNAsync", 3)
    }
}
```

**方法 ②**

导入模块 `mapActions`

```javascript
import { mapActions } from "vuex";
```

将需要的函数映射为当前组件的 methods

```javascript
methods: {
    ...mapActions( ['addAsync', 'addNAsync'] )
    addBtnAsync() {
        this.addAsync()
    }
    addNBtnAsync() {
        this.addNasync(3)
    }
}
```

> 也可以直接使用，不用另外再在页面声明一个事件处理函数来调用

直接调用 `...mapActions` 映射过来的方法

```html
<button @click="addNAsync(3)">加N</button>
```

## Getter

有时候我们需要从 store 中的 state 中派生出一些状态(数据)，getter 就是对 store 中的数据进行加工后形成新的数据，类似于**计算属性**

store 中的数据发生变化，getter 的数据也会跟着变化

### 定义

在 Vuex 的入口文件中

```javascript
const store = new Vuex.Store({
  state: {
    count: 0,
  },
  getters: {
    showNum: function (state) {
      return "当前的数值是" + state.count;
    },
  },
});
```

### 触发

**方法 ①**

```javascript
this.$store.getters.名称;
```

**方法 ②**

使用 `mapGetters`

```javascript
import { mapGetters } from "vuex";
```

映射在当前的计算属性中

```javascript
computed: {
    ...mapGetters( ['showNum'] )
}
```

例如

```html
<!--正常使用数据-->
<h3>当前的数值是：{{ $store.state.count }}</h3>
<!--使用 mapGetters 的时候-->
<h3>{{ showNum }}</h3>
```

# 案例代码

<a href="https://github.com/dsjerry/Vuex-">Github</a>
