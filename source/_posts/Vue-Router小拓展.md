---
title: Vue Router知识小补充
date: 2020-08-05 09:17:50
tags: vue
categories: 前端学习
thumbnail: https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=3098602264,1250854698&fm=26&gp=0.jpg
toc: true
---

在这里做些小笔记，一边看笔记一边看官方文档，方便理解

文章作为提纲，配合官方文档学习<!-- more -->

前提小知识点：

通过注入路由器，我们可以在任何组件内通过 `this.$router` 访问路由器，也可以通过 `this.$route` 访问当前路由

嵌套路由要嵌套的内容也是写进模板组件里面的，而不是写进本身的HTML里面。然后需要在 `VueRouter` 的参数中使用 `children`

**路由组件传参**，就是使用 `props` 来代替 `$route.xx.xx`，提高灵活度

# 基础标签

## router-link

```html
<router-link to="/foo">Go to Foo</router-link>
```

`<router-link>` 组件支持用户在具有路由功能的应用中 (点击) 导航。 通过 `to` 属性指定目标地址，默认渲染成带有正确链接的 `<a>` 标签，可以通过配置 `tag` 属性生成别的标签.。

> 当点击`<router-link>` 的时候，其实可以说等同于调用了`router.push(...)`，这个东西就是**编程式的导航**，router-link 是**声明式**

当 `<router-link>` 对应的路由匹配成功，将自动设置 class 属性值 `.router-link-active`。

这个 class 也可以自定义，通过路由的构造选项 `linkActiveClass` 来全局配置：

```javascript
var router = new VueRouter({
	routes,
	linkActiveClass: 'active',
});
```

- 然后在style配置class样式就行

## router-view

`<router-view>` 是一个 functional 组件，渲染路径匹配到的视图组件，它里面可以内嵌自己的 `<router-view>`

可以添加动态效果，使用 `<transition>` 组件：

```html
<transition mode="out-in">
  <router-view></router-view>
</transition>
```

- `out-in` ：当前元素先进行过渡，完成之后新元素过渡进入。
- `in-out` ：新元素先进行过渡，完成之后当前元素过渡离开。

使用的时候要设置相对应的 css 样式，例如：

```css
.v-enter,
.v-leave-to {
	opacity: 0;
	transform: translateX(150px);
}
.v-enter-active,
.v-leave-active {
	transition: all 1s ease;
}
```



# 路由对象

一个**路由对象 (route object)** 表示当前激活的路由的状态信息，包含了当前 URL 解析得到的信息，还有 URL 匹配到的**路由记录 (route records)**。

**例如**：对于 `$route.params`：

|             模式              |      匹配路径       |             $route.params              |
| :---------------------------: | :-----------------: | :------------------------------------: |
|        /user/:username        |     /user/evan      |         `{ username: 'evan' }`         |
| /user/:username/post/:post_id | /user/evan/post/123 | `{ username: 'evan', post_id: '123' }` |

### 路由对象属性

- **$route.path**

  - 类型: `string`

    字符串，对应当前路由的路径，总是解析为绝对路径，如 `"/foo/bar"`。

- **$route.params**

  - 类型: `Object`

    一个 key/value 对象，包含了动态片段和全匹配片段，例如，对于路径 `/login/13`，则有 `$route.params.id == 13`，如果没有路由参数，就是一个空对象。

- **$route.query**

  - 类型: `Object`

    一个 key/value 对象，表示 URL 查询参数。例如，对于路径 `/foo?user=1`，则有 `$route.query.user == 1`，如果没有查询参数，则是个空对象。

- ......

# API

## vm.$mount()

- 如果 Vue 实例在实例化时没有收到 el 选项，则它处于“未挂载”状态，没有关联的 DOM 元素。可以使用     vm.$mount() 手动地挂载一个未挂载的实例。

- 返回值是 vm 实例自身

  ```javascript
  const app = new Vue({
     router
  }).$mount('#app')
  ```

  