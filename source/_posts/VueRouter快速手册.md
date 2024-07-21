---
title: VueRouter快速手册
toc: true
date: 2023-08-26 15:27:20
tags: [vue]
categories: 前端学习
thumbnail: https://z1.ax1x.com/2023/09/03/pPDZAPO.jpg
cover: https://z1.ax1x.com/2023/09/03/pPDZAPO.jpg
---

🫡🙃

[ ] 先看官方文档，然后组织出来笔记和案例代码

<!-- more -->

# 安装

```shell
npm install vue-router@4
```

# 基础

## 两个标签

### router-link

跳转没用`<a>`标签，用`<router-link>`标签，在不重新加载页面的情况下，改变 URL

标签属性：

- to：指定跳转的路径，最后编译成`<a>`标签的`href`属性
- tag：指定渲染成什么标签，默认是`<a>`
- replace：默认是`false`，点击跳转后，会向 history 添加一个新的记录，设置为`true`，则不会添加新记录，替换当前的 history 记录
- active-class：指定当前路由高亮的类名，默认是`router-link-active`

除了使用标签来导航(**声明式导航**)，还可以使用`router.push`方法，**编程式导航**

### router-view

- 路由匹配到的组件将渲染在这里
- 一个页面中可以有多个`<router-view>`标签，用于显示不同的内容

## 创建路由

> 官网推荐的是通过动态导入组件来实现创建路由(路由懒加载)

📄router/index.ts: 创建路由

```typescript
import { createRouter, createWebHistory, RouteRecordRaw } from "vue-router";
import Home from "../views/Home.vue";
import About from "../views/About.vue";

const routes: RouteRecordRaw[] = [
  {
    path: "/",
    name: "Home",
    component: Home,
  },
  {
    path: "/about",
    name: "About",
    component: About,
  },
  {
    path: "/profle",
    name: "Profile",
    component: () => import("../views/Profile.vue"), // 动态加载
  },
];

const router = createRouter({
  history: createWebHistory(),
  routes,
});

export default router;
```

📄main.ts: 将路由当成插件挂在到 Vue 实例中

```typescript
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";

createApp(App).use(router).mount("#app");
```

📄Home.vue：两个标签，实现路由跳转和渲染

```html
<template>
  <div>
    <h1>Home</h1>
    <router-link to="/about">About</router-link>
    <router-view></router-view>
  </div>
</template>
```

📄About.vue：在*setup*中通过 hooks 实现跳转、获取路由参数

```html
<template>
  <div>
    <h1>About</h1>
    <router-link to="/">Home</router-link>
    <router-view></router-view>
  </div>
</template>
<script setup lang="ts">
  import { useRouter, useRoute } from "vue-router";
  const router = useRouter(); // options API 中的 this.$router
  const route = useRoute(); // options API 中的 this.$route

  function pushWithQuery(query) {
    router.push({
      name: "home",
      query: {
        ...route.query,
        ...query,
      },
    });
  }
</script>
```

- 在模板中可以访问到 $router 和 $route 对象，但是在 setup 中不能访问，需要通过`useRouter`和`useRoute`获取

> `router` 和 `route` 的区别：router 是路由实例，route 是当前路由对象

> 在`setup`中使用的时候，`useRouter`和`useRoute`必须是顶级调用，如果在方法里面调用，得到的返回值是`undefined`

router 的一些方法：

- `router.push`：跳转到指定路由
- `router.replace`：跳转到指定路由，不会在 history 中添加新记录
- `router.go`：前进或后退指定步数
- `router.back`：后退
- `router.forward`：前进
- `router.addRoute`：**动态添加路由**
- `router.removeRoute`：**动态删除路由**
- `router.getRoutes`：获取所有路由，是一个数组
- `router.hasRoute`：判断是否有指定路由

route 对象是响应式的，可以通过`watch`监听变化

```typescript
import { useRoute } from "vue-router";
import { ref, watch } from "vue";

const route = useRoute();
const userData = ref();

// 当参数更改时获取用户信息
watch(
  () => route.params.id,
  async (newId) => {
    userData.value = await fetchUser(newId);
  }
);
```

### 动态路由

> `router.addRoute`只注册一个新路由，如果新增的路由与当前位置匹配，就得手动配置`router.push`或者`router.replace`跳转

```js
router.addRoute({ path: "/about", component: About });
// 我们也可以使用 this.$route 或 route = useRoute() （在 setup 中）
router.replace(router.currentRoute.value.fullPath);
```

- 如果需要等待新的路由显示，可以使用 `await router.replace()`

在导航守卫中添加路由的时候，使用`return`返回新的路由，而不是使用`router.push`或者`router.replace`

```js
router.beforeEach((to) => {
  if (!hasNecessaryRoute(to)) {
    router.addRoute(generateRoute(to));
    // 触发重定向
    return to.fullPath;
  }
});
```

- `hasNecessaryRoute()` 在添加新的路由后返回 `false`，以避免无限重定向。

添加*嵌套路由*

```js
router.addRoute({ name: "admin", path: "/admin", component: Admin });
router.addRoute("admin", { path: "settings", component: AdminSettings });
```

🗑️ 删除路由有几个方式:

1. 直接使用`router.addRoute`，如果有重名的，先删除再添加

```js
router.addRoute({ path: "/about", name: "about", component: About });
// 这将会删除之前已经添加的路由，因为他们具有相同的名字且名字必须是唯一的
router.addRoute({ path: "/other", name: "about", component: Other });
```

2. 通过`router.addRoute()`返回的回调函数来删除

```js
const removeRoute = router.addRoute(routeRecord);
removeRoute(); // 删除路由如果存在的话
```

3. 通过`router.removeRoute`来按*名称*删除

```js
router.addRoute({ path: "/about", name: "about", component: About });
// 删除路由
router.removeRoute("about");
```

### 数据获取

进入路由之后需要从服务器获取数据，一般两种方式：

- 导航完成之后获取，先导航，然后在组件的声明周期函数中获取。这期间显示 _正在加载_ z 之类的动画，然后也可以添加个`watch`以便再次获取
- 导航完成之前获取，在导航守卫中进行

## 编程式导航

🌎 这个方法会向 history 栈添加一个新的记录，所以，当用户点击浏览器后退按钮时，会回到之前的 URL。

- 点击导航标签，内部会调用`router.push`方法

`push`方法的参数可以是一个字符串，也可以是一个描述地址的对象

```typescript
// 字符串路径
router.push("/users/eduardo");

// 带有路径的对象
router.push({ path: "/users/eduardo" });

// 命名的路由，并加上参数，让路由建立 url
router.push({ name: "user", params: { username: "eduardo" } });

// 带查询参数，结果是 /register?plan=private
router.push({ path: "/register", query: { plan: "private" } });

// 带 hash，结果是 /about#team
router.push({ path: "/about", hash: "#team" });
```

### 动态路由匹配

- 目的是为了根据不同的参数，渲染不同的内容

动态路由匹配的参数，可以通过`useRoute`获取

- `useRoute`返回的是响应式的对象，所以可以通过`watch`监听变化
- 也可以通过`$route.params`获取，`$route`不是响应式的

除了`params`，还有`query`、`hash`等，`params`和`query`都是对象，`hash`是字符串

```js
const User = {
  template: "<div>User</div>",
};

// 这些都会传递给 `createRouter`
const routes = [
  // 动态字段以冒号开始
  { path: "/users/:id", component: User },
];
```

可以传递多个参数，它们会映射到 `$route.params` 上的相应字段

|            匹配模式            |         匹配路径         |             $route.params              |
| :----------------------------: | :----------------------: | :------------------------------------: |
|        /users/:username        |      /users/eduardo      |        { username: 'eduardo' }         |
| /users/:username/posts/:postId | /users/eduardo/posts/123 | { username: 'eduardo', postId: '123' } |

> 路由参数变化，组件不会重新渲染，因为组件的复用，可以通过`watch`监听参数变化，重新获取数据从而更新组件。或者使用导航守卫监听参数变化

```typescript
const User = {
  template: "...",
  async beforeRouteUpdate(to, from) {
    // 对路由变化做出响应...
    this.userData = await fetchUser(to.params.id);
  },
};
```

#### 使用正则匹配

常规匹配的时候，内部会使用`([^/]+)`来匹配参数

- 至少有一个字符不是`/`

如果想根据参数的格式来匹配，可以使用自定义正则表达式

```ts
const routes = [
  // /:orderId -> 仅匹配数字
  { path: "/:orderId(\\d+)" },
  // /:productName -> 匹配其他任何内容
  { path: "/:productName" },
];
```

- `\d`需要使用`\\d`来转义

#### 可重复的参数

将参数标记为可重复：

- 通过`+`来表示参数可以重复，可以是一个或多个
  - 就是说至少有一个字符不是`/`
- 通过`*`来表示参数可以重复，可以是零个或多个
  - 就是说可以是空字符串，也就是可选的
  - 通过`?`也可以表示可选的

```ts
const routes = [
  // /:chapters ->  匹配 /one, /one/two, /one/two/three, 等
  { path: "/:chapters+" },
  // /:chapters -> 匹配 /, /one, /one/two, /one/two/three, 等
  { path: "/:chapters*" },
  // 可选
  { path: "/:chapters?" },
];
```

在传递的时候，对应的参数需要传递数组

```ts
// 给定 { path: '/:chapters*', name: 'chapters' },
router.resolve({ name: "chapters", params: { chapters: [] } }).href;
// 产生 /
router.resolve({ name: "chapters", params: { chapters: ["a", "b"] } }).href;
// 产生 /a/b

// 给定 { path: '/:chapters+', name: 'chapters' },
router.resolve({ name: "chapters", params: { chapters: [] } }).href;
// 抛出错误，因为 `chapters` 为空
```

- `router.resolve`方法可以根据路由名称和参数，返回一个 URL

同时可以加上正则表达式

```ts
const routes = [
  // 仅匹配数字
  // 匹配 /1, /1/2, 等
  { path: "/:chapters(\\d+)+" },
  // 匹配 /, /1, /1/2, 等
  { path: "/:chapters(\\d+)*" },
];
```

#### Sensitive 和 strict 路由

例如，路由 `/users` 将匹配 `/users`、`/users/`、甚至 `/Users/`

可通过全局配置配置：

```ts
const router = createRouter({
  routes,
  sensitive: true, // 匹配规则是否大小写敏感？(默认值：false)
  strict: true, // 匹配规则是否严格模式？(默认值：false)
});
```

也可以在路由配置中配置：

```ts
const router = createRouter({
  history: createWebHistory(),
  routes: [
    // 将匹配 /users/posva 而非：
    // - /users/posva/ 当 strict: true
    // - /Users/posva 当 sensitive: true
    { path: "/users/:id", sensitive: true },
    // 将匹配 /users, /Users, 以及 /users/42 而非 /users/ 或 /users/42/
    { path: "/users/:id?" },
  ],
  strict: true, // applies to all routes
});
```

### 嵌套路由

一个页面中有多个`<router-view>`标签，用于显示不同的内容

- 最外层的`<router-view>`标签，用于显示最外层的内容，也就是父路由的内容
- 一个被渲染的组件里面也可以有自己的`<router-view>`标签，用于显示内层的内容，也就是子路由的内容

```js
const routes = [
  {
    path: "/user/:id",
    component: User,
    children: [
      {
        // 当 /user/:id/profile 匹配成功
        // UserProfile 将被渲染到 User 的 <router-view> 内部
        path: "profile",
        component: UserProfile,
      },
      {
        // 当 /user/:id/posts 匹配成功
        // UserPosts 将被渲染到 User 的 <router-view> 内部
        path: "posts",
        component: UserPosts,
      },
    ],
  },
];
```

- 以 `/` 开头的嵌套路径将被视为根路径

如果访问了不存在的路由，`<router-view>`里面什么都没有，可以通过`<router-view v-slot="{ Component }">`来处理

```html
<router-view v-slot="{ Component }">
  <div v-if="Component">
    <!-- 只有在匹配到的路由有组件时，才会显示 -->
    <transition name="fade" mode="out-in">
      <component :is="Component" />
    </transition>
  </div>
</router-view>
```

也可以提供一个空的嵌套路由作为默认子路由（path 传递空字符串）

```js
const routes = [
  {
    path: "/user/:id",
    component: User,
    children: [
      // 当 /user/:id 匹配成功
      // UserHome 将被渲染到 User 的 <router-view> 内部
      { path: "", component: UserHome },

      // ...其他子路由
    ],
  },
];
```

## 命名路由和命名视图

### 命名路由

给路由配置一个`name`属性，可以通过`name`来跳转

- 没有硬编码的 URL，没有硬编码的 URL，所以如果想改变 /user/:id，可以随时修改路由配置，而不用在代码中搜索所有用到该路径的地方
- params 的自动编码/解码。
- 防止你在 url 中出现打字错误。
- 绕过路径排序（如显示一个）/user/:id 和 /user/:username，因为后者是在前者之后定义的。

params 的自动编码/解码，更方便使用参数

```js
const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: "/user/:userId",
      name: "user",
      component: User,
    },
  ],
});

// 通过命名路由跳转
router.push({ name: "user", params: { userId: "123" } });

// 带查询参数，变成 /register?plan=private
router.push({ name: "register", query: { plan: "private" } });
```

或者在标签中使用

```html
<router-link :to="{ name: 'user', params: { username: 'erina' }}">
  User
</router-link>
```

### 命名视图

给`<router-view>`标签添加`name`属性，可以在路由配置中指定渲染的组件。例如创建一个布局，有 sidebar (侧导航) 和 main (主内容) 两个视图

```html
<router-view class="view left-sidebar" name="LeftSidebar"></router-view>
<router-view class="view main-content"></router-view>
<router-view class="view right-sidebar" name="RightSidebar"></router-view>
```

这个路由中有多个视图，所以得使用`components`配置

```js
const router = createRouter({
  history: createWebHashHistory(),
  routes: [
    {
      path: "/",
      components: {
        default: Home,
        // LeftSidebar: LeftSidebar 的缩写
        LeftSidebar,
        // 它们与 `<router-view>` 上的 `name` 属性匹配
        RightSidebar,
      },
    },
  ],
});
```

## 重定向和别名

### 重定向

通过 routes 配置来完成

```js
const routes = [{ path: "/home", redirect: "/" }];
```

也可以传递一个命名的路由

```js
const routes = [{ path: "/home", redirect: { name: "home" } }];
```

甚至可以传递一个方法，动态返回重定向的目标

```js
const routes = [
  {
    // /search/screens -> /search?q=screens
    path: "/search/:searchText",
    redirect: (to) => {
      // 方法接收目标路由作为参数
      // return 重定向的字符串路径/路径对象
      return { path: "/search", query: { q: to.params.searchText } };
    },
  },
  {
    path: "/search",
    // ...
  },
];
```

> 重定向时可以不指定`component`，因为重定向的时候，没有组件需要渲染

**相对重定向**

- 重定向的目标也可以是相对路径，这样就不用写完整的路径了

```js
const routes = [
  {
    // 将总是把/users/123/posts重定向到/users/123/profile。
    path: "/users/:id/posts",
    redirect: (to) => {
      // 该函数接收目标路由作为参数
      // 相对位置不以`/`开头
      // 或 { path: 'profile'}
      return "profile";
    },
  },
];
```

### 别名

- 重定向的时候`/home`会替换掉`/`，别名不会改变 URL，但是会在路由匹配时添加到 history 记录中

所以应该这样配置

```js
const routes = [{ path: "/", component: Homepage, alias: "/home" }];
```

通过别名可以将 UI 结构映射到一个任意的 URL，而不受配置的嵌套结构的限制。使别名以 / 开头，以使嵌套路径中的路径成为绝对路径。你甚至可以将两者结合起来，用一个数组提供多个别名：

```js
const routes = [
  {
    path: "/users",
    component: UsersLayout,
    children: [
      // 为这 3 个 URL 呈现 UserList
      // - /users
      // - /users/list
      // - /people
      { path: "", component: UserList, alias: ["/people", "list"] },
    ],
  },
];
```

## 路由组件传参

通过使用 props 将路由参数传递给组件

> 在组件中获取路由传过来的 _props_ 时，需要定义然后使用

**布尔模式**：将`props`设置为`true`，`route.params`会被设置为组件的 props

```js
const User = {
  template: "<div>User {{ $route.params.id }}</div>",
};
const routes = [{ path: "/user/:id", component: User }];
```

替换成

```js
const User = {
  // 请确保添加一个与路由参数完全相同的 prop 名
  props: ["id"],
  template: "<div>User {{ id }}</div>",
};
const routes = [{ path: "/user/:id", component: User, props: true }];
```

**命名视图**：每个视图需要定义自己的 props

```js
const routes = [
  {
    path: "/user/:id",
    components: { default: User, sidebar: Sidebar },
    props: { default: true, sidebar: false },
  },
];
```

**对象模式**：如果 props 被设置为一个对象，它会被按原样设置为组件的 props

```js
const routes = [
  {
    path: "/user/:id",
    components: { default: User, sidebar: Sidebar },
    props: { default: true, sidebar: false },
  },
];
```

**函数模式**：可以创建一个函数返回 props，将参数转换为其他类型，将静态值与基于路由的值相结合等等。

```js
const routes = [
  {
    path: "/search",
    component: SearchUser,
    props: (route) => ({ query: route.query.q }),
  },
];
```

## 路由元信息

配置`meta`

```js
const routes = [
  {
    path: "/",
    component: Home,
    // 路由元信息
    meta: { requiresAuth: true },
  },
];
```

- 数组中每个对象叫做**路由记录**，在嵌套路由中就分为**父路由记录**和**子路由记录**
- 匹配到的路由暴露为`$route`对象的`matched`属性，是一个数组，包含了所有匹配到的路由记录

`$route.meta`方法可以获取到路由元信息，直接用`.`访问

```js
router.beforeEach((to, from) => {
  // 而不是去检查每条路由记录
  // to.matched.some(record => record.meta.requiresAuth)
  if (to.meta.requiresAuth && !auth.isLoggedIn()) {
    // 此路由需要授权，请检查是否已登录
    // 如果没有，则重定向到登录页面
    return {
      path: "/login",
      // 保存我们所在的位置，以便以后再来
      query: { redirect: to.fullPath },
    };
  }
});
```

`TypeScript`中，可以通过`RouteMeta`来定义路由元信息的类型

```ts
// typings.d.ts or router.ts
import "vue-router";

declare module "vue-router" {
  interface RouteMeta {
    // 是可选的
    isAdmin?: boolean;
    // 每个路由都必须声明
    requiresAuth: boolean;
  }
}
```

> `props`会提供这个吗以后 ？？🫡

## 不同的历史模式

在以往*vue2*使用的时候，使用`mode`来配置，*vue3*现在使用`history`来配置

它们之间的对应关系：

- mode: history | hash | abstract
- history: createWebHistory | createWebHashHistory | createMemoryHistory

**👉 hash 模式**

在内部传递实际 URL 之前使用了一个哈希字符（`#`）

URL 看起来就像这样：`http://oursite.com/#/user/id`

它的好处是不需要后端配置，缺点是**在 SEO 方面不太友好**

**👉 HTML5 模式**

✅ 推荐使用

URL 开起来就像这样：`http://oursite.com/user/id`

它的好处是 URL 看起来更加友好，但是需要后端配置

# 进阶

## 导航守卫

导航守卫是一个函数，它会在路由发生变化时被调用

出现的地方有：全局的、单个路由独享的、组件内的

### 全局前置守卫

`router.beforeEach`方法注册一个全局前置守卫，守卫是异步的，所以跳转需要在守卫`resolve`之后才会继续

```js
const router = createRouter({ ... })

router.beforeEach((to, from) => {
  // ...
  // 返回 false 以取消导航
  return false
})
```

回调参数：

- `to`：即将要进入的目标路由对象
- `from`：当前导航正要离开的路由
- `next`：调用该方法后，能进入下一个钩子，不过现在可以用`return`来代替`next`
  - `next(false)`：取消当前导航
  - `next('/')`：跳转
  - `next({ path: '/' })`：跳转

现在 _Vue Router 4_ 中可以使用`return`，不用`next()`，返回值可以有：

- `false`：取消当前导航
- 一个路由地址：跳转到指定路由，类似于`router.push()`，也可以设置`replace: true`、`name: 'home'`等
- 如果什么都没有，`undefined` 或返回 `true`，则导航是有效的，并调用下一个导航守卫

> return 会比 next 好点吧应该，因为 next 必须得调用，而 return 可以不用写那么多 if else

如果遇到了意料之外的情况，可能会抛出一个 Error。这会取消导航并且调用 `router.onError()` 注册过的回调。

```js
router.beforeEach(async (to, from) => {
  if (
    // 检查用户是否已登录
    !isAuthenticated &&
    // ❗️ 避免无限重定向
    to.name !== "Login"
  ) {
    // 将用户重定向到登录页面
    return { name: "Login" };
  }
});
```

### 全局解析守卫

`router.beforeResolve`方法注册一个全局解析守卫，和前置导航守卫类似，但是得在 **所有组件内守卫**和 **异步路由组件** 被解析之后，解析守卫就被调用

示例：确保用户可访问到自定义*meta*

```js
router.beforeResolve(async (to) => {
  if (to.meta.requiresCamera) {
    try {
      await askForCameraPermission();
    } catch (error) {
      if (error instanceof NotAllowedError) {
        // ... 处理错误，然后取消导航
        return false;
      } else {
        // 意料之外的错误，取消导航并把错误传给全局处理器
        throw error;
      }
    }
  }
});
```

router.beforeResolve 是获取数据或执行任何其他操作（如果用户无法进入页面时你希望避免执行的操作）的理想位置。

### 全局后置钩子

- 和守卫不同的是，后置钩子没有 `next` 函数，不能改变导航本身；第三个参数可以是`failure`
- 对于分析、更改页面标题、声明页面等辅助功能以及许多其他事情都很有用。

```js
router.afterEach((to, from, failure) => {
  if (!failure) sendToAnalytics(to.fullPath);
});
```

也就是**导航故障**

## 导航故障

### 异步等待导航结果

导航故障是导航期间出现的任何错误或被取消的导航。当使用 `router-link` 组件时，Vue Router 会自动调用 `router.push` 来触发一次导航。

一般来说导航都会到一个新的页面，想要在导航完成之后做一些事情，可以使用`router.afterEach`，但是如果导航被取消了（或者错误之类的），就不会触发`router.afterEach`

解决的办法就是使用**导航的异步特性**进行等待

比如等导航结束之后再关闭菜单

```js
await router.push("/my-profile");
this.isMenuOpen = false;
```

或者是监听导航故障，然后做点什么

```js
router.onError((error) => {
  if (error instanceof NotAllowedError) {
    // ...
  }
});
```

### 故障

导航成功的话，`await`等到的是`undefined`，如果导航失败了，`await`等到的是一个`NavigationFailure`对象

```js
const navigationResult = await router.push("/my-profile");

if (navigationResult) {
  // 导航被阻止
} else {
  // 导航成功 (包括重新导航的情况)
  this.isMenuOpen = false;
}
```

或者是(真的吗)

```js
try {
  await router.push("/my-profile");
  this.isMenuOpen = false;
} catch (error) {
  if (error instanceof NotAllowedError) {
    // ...
  } else {
    // 意料之外的错误，取消导航并把错误传给全局处理器
    throw error;
  }
}
```

`**isNavigationFailure()**`：来了解哪些导航被阻止了和为什么被阻止

```js
import { NavigationFailureType, isNavigationFailure } from "vue-router";

// 试图离开未保存的编辑文本界面
const failure = await router.push("/articles/2");

if (isNavigationFailure(failure, NavigationFailureType.aborted)) {
  // 给用户显示一个小通知
  showToast("You have unsaved changes, discard and leave anyway?");
}
```

👉 `isNavigationFailure()` 第二个参数是一个字符串，可以是

- `aborted`：导航被取消，`return false`或`next(false)`
- `cancelled`：当前导航没完成之前，又触发了一个新的导航（在导航等待的时候，调用了个新的`router.push()`）
- `duplicated`：导航被阻止，因为已经在目标位置了

如果第二个参数不传的话，就只是判断是否是*Navigation Failure* 🙃

👉 `isNavigationFailure()` 第一个参数，记录着路由的`to`, `from`

### 检测重定向

重定向不会阻止导航，而是创建一个新的导航，所以`router.push()`返回的是`undefined`

```js
await router.push("/my-profile");
if (router.currentRoute.value.redirectedFrom) {
  // redirectedFrom 是解析出的路由地址，就像导航守卫中的 to和 from
}
```

## 过渡特效和滚动行为

### 过度特效

```html
<router-view v-slot="{ Component }">
  <transition name="fade">
    <component :is="Component" />
  </transition>
</router-view>
```

- `Component`是当前路由匹配到的组件
- `v-solt`可以用简写`#default="{ Component }"`

除了使用`v-slot`，还可以使用`<router-view>`的`transition`属性

```html
<router-view transition="fade" />
```

🙃 针对每个路由配置不同的过渡特效

- 可以在路由配置中加入`meta`
- 或者在全局后置钩子中加入`meta`

```js
const routes = [
  {
    path: "/custom-transition",
    component: PanelLeft,
    meta: { transition: "slide-left" },
  },
  {
    path: "/other-transition",
    component: PanelRight,
    meta: { transition: "slide-right" },
  },
];
```

```html
<router-view v-slot="{ Component, route }">
  <!-- 使用任何自定义过渡和回退到 `fade` -->
  <transition :name="route.meta.transition || 'fade'">
    <component :is="Component" />
  </transition>
</router-view>
```

❔❔ Vue 可能会自动复用看起来相似的组件，从而忽略了任何过渡。可以给动态组件加上一个 `key` 属性，来提示 Vue 去强制重新渲染

```html
<router-view v-slot="{ Component, route }">
  <transition name="fade">
    <component :is="Component" :key="route.path" />
  </transition>
</router-view>
```

### 滚动行为

- 就是在前进后退的时候，页面滚动到哪里，或者是保持原来的滚动位置

> 注意: 这个功能只在支持 history.pushState 的浏览器中可用。

```js
const router = createRouter({
  history: createWebHistory(),
  scrollBehavior(to, from, savedPosition) {
    // return 期望滚动到哪个的位置
  },
});
```

- 返回一个*falsy*值，或者是一个空对象，不会发生滚动
- 返回`savedPosition`，会滚动到之前保存的位置

位置使用的是`top`和`left`，可以是数字或者是字符串

```js
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    // 始终在元素 #main 上方滚动 10px
    return {
      // 也可以这么写
      // el: document.getElementById('main'),
      el: "#main",
      top: -10,
    };
  },
});
```

如果你要模拟 “滚动到锚点” 的行为：

```js
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    if (to.hash) {
      return {
        el: to.hash,
        behavior: "smooth", // 滚得更流畅，如果浏览器支持的话
      };
    }
  },
});
```

延迟滚动，等待页面动效完成后再滚动，可以返回个`Promise`

```js
const router = createRouter({
  scrollBehavior(to, from, savedPosition) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve({ left: 0, top: 0 });
      }, 500);
    });
  },
});
```

<!-- <br />
<br />
<br />
<br />
<div style="text-align: center;">
<h1 style="font-size: 50px; margin-bottom: -20px;"><span style="color: #41b883;">VueRouter</span  > 快速手册 <span style="font-size: 20px; margin-left: -10px";margin-top: -20px;>⚡</span></h1>
<h1> 🚄 🚅 🚈 🚝 💨</h1>
</div>

<br />
<br />
<br />
<br /> -->
